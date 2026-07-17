# OrgA

An AI agent that lives entirely inside a single Emacs Org-mode file.

> **AI-assisted disclosure:** This README was written by Claude (Anthropic), based on a read-through of `orga.org`. Please review it against the source before trusting it as documentation.

## What is this?

OrgA collapses an entire LLM agent — runtime, tool definitions, system prompt, and chat history — into one `.org` file. There's no server, no separate config, no database. You open `orga.org`, explicitly execute its initialization block, and from then on the conversation *is* the document: every turn is appended as Org headings, and every reply is written back into the buffer for you to read, edit, or fork with normal Org-mode tools.

It talks to any [OpenRouter](https://openrouter.ai)-compatible model. Opening an OrgA document is safe by default: it never executes source blocks automatically.

## How it works

After you explicitly execute the `orga__initialize` source block, Emacs loads a small pipeline:

**Collect → Assemble → Send → Parse → Commit → Execute → Yield**

1. **Collect** — reads the system prompt, active goal, running summary, and full chat history out of the buffer.
2. **Assemble** — concatenates those into the message array sent to the API, in a configurable order (`orga/context-slot-order`).
3. **Send** — resolves the API key and POSTs to OpenRouter.
4. **Parse** — decodes the response (content, tool calls, reasoning, token usage, cost).
5. **Commit** — writes the result back into the buffer as a new `** Assistant` (or `** Assistant Tool Call`) heading, before anything is executed.
6. **Execute** — runs any `#+CALL:` lines the model requested (manually by default, or automatically if you opt in).
7. **Yield** — decides whether to loop (more tool calls / goal not yet met) or stop and wait for the next `** Input`.

Everything is a plain heading, property, or source block, so the whole state of the agent — including its memory — is just Org-mode text you can inspect, diff, and version-control.

## Key concepts

- **Chat History** — lives under `* Chat History` as `** User`, `** Assistant`, `** Assistant Tool Call`, and `** Tool` headings. Reasoning traces are kept in `:REASONING:...:END:` drawers and are stripped before being sent back to the API, so chain-of-thought never leaks into future context.
- **Tools** — declared under `* Tools`. Each tool subtree bundles three artifacts: a human-readable JSON schema, an Emacs Lisp alist schema (`orga_tool_schema__<name>`), and an implementation block (`orga_tool_implementation__<name>`). Tag a heading `:disabled:` to pull a tool from the API call without deleting it. Ships with three examples: `shell`, `web_search`, and `fetch` (plus a deliberately unimplemented `conquer_the_world` as a demo of the toggle).
- **Goal Mode** — an optional `* Goal` section describing a completion condition, either a deterministic shell check or a fuzzy one judged by the LLM. When active, OrgA keeps looping turns on its own, logging each evaluation under `* Goal Log`, until the goal is met or a turn cap is hit.
- **Compaction** — once token usage crosses a threshold (`orga/compaction-threshold`, default 50% of the model's context window), the oldest turns are summarized into a cumulative `* Summary` block and a `:cutoff:` tag advances past them, keeping the N most recent turns live (`orga/keep-recent-turns`).
- **Dry Run** — toggle to preview the exact payload that would be sent, without making a live call.
- **Wiki** — a `* Wiki` section of `[[wiki://...]]` -linked Markdown notes the agent can browse via the `fetch` tool, in the spirit of an "LLM wiki" / SOUL.md-style knowledge base.
- **System Prompt** — a single Markdown block (also tangled out to `SOUL.md`) that defines the agent's identity, its runtime, and its behavioral rules — including "never start a line with `*`," since that would be misread as a new Org heading.

## Setup

1. Provide an OpenRouter API key one of three ways (checked in this order):
   - Paste it into the `openrouter_key` block under `* LLM Configurations`.
   - Add `OPENROUTER_API_KEY=...` to a `.env` file next to `orga.org`.
   - Export `OPENROUTER_API_KEY` in your shell environment.
2. Set the model you want in the `openrouter_model` block (defaults to `google/gemini-3.5-flash`).
3. Open `orga.org` in Emacs. **Opening it does not run any code.**
4. Explicitly execute the initialization block:
   - Run `M-x org-babel-goto-named-src-block RET orga__initialize RET`.
   - With point in the block, press `C-c C-c` and confirm evaluation if Emacs asks.

   Emacs reports `OrgA loaded -- C-c C-v S to send.` when the runtime is ready.

To initialize OrgA noninteractively, use an explicit batch invocation:

```sh
emacs --batch /path/to/orga.org \
  --eval '(progn
            (require (quote org))
            (require (quote ob-core))
            (with-current-buffer (get-file-buffer "/path/to/orga.org")
              (goto-char (org-babel-find-named-block "orga__initialize"))
              (let ((org-confirm-babel-evaluate nil))
                (org-babel-execute-src-block)))))'
```

Only execute the initialization block in an OrgA document whose source you trust; it evaluates that document's runtime.

## Usage

Write your message under the trailing `** Input` heading, then send it.

| Action | Local (`C-c C-v …`) | Global |
|---|---|---|
| Send | `S` | `<f9>` |
| Toggle dry-run | `D` | `<f8>` |
| Toggle goal mode | `G` | `<f5>` |
| Execute pending `#+CALL:` lines | `E` | `<f7>` |
| Compact context now | `C` | `<f6>` |

When the model calls a tool, OrgA scaffolds a `** Tool` heading with a `#+CALL:` line per call. By default you run each with `C-c C-c` and re-send; set `orga/auto-eval-p` to `t` if you want tool calls executed and looped automatically.

⚠️ **Auto-eval + shell tool is dangerous.** The `shell` tool gives the model direct command execution. Turning on `orga/auto-eval-p` while it's enabled means the model can run arbitrary commands on your machine without you looking at them first. The code itself warns: if you do this, you're accepting that risk.

## Project structure (inside `orga.org`)

```
* LLM Configurations         API key / model
* Initialize AI Sender Method  the runtime (Collect, Assemble, Send, Parse,
                                Commit, Execute, Yield, Compaction, Goal,
                                Cycle Driver, Composition)
* Wiki                       browsable concept notes
* Tools                      shell, web_search, fetch, (disabled) conquer_the_world
* System Prompt              agent identity, tangled to SOUL.md
* Goal / Goal 2 / Goal Log   optional autonomous-loop objective + history
* Summary                    rolling compacted memory
* Chat History               the live conversation
```

## Status / caveats

- Tool execution is manual by default and involves the human running `#+CALL:` lines — treat this as a deliberate safety pause, not a bug.
- The `conquer_the_world` tool is an intentionally-unimplemented joke/demo showing how the `:disabled:` tag and a stub Python implementation work together.
- This is a single-file, Emacs-only project; there's no packaging step described here beyond opening the file.

