- [ ] implement RAG
  - [X] refactor `* System Prompt`
  - [X] refactor how we read `* System Prompt`(read only the md block)
  - [X] add wiki section
  - [ ] create a fetch tool that can handle:
    - [ ] "wiki://path/to/concept"
        - [ ] search in file under `* Wiki` header
        - [ ] fallback to search in `wiki/` directory
        - [ ] on fail suggest fetch again with concept name(freeform text)
        - [ ] print to tool response
    - [ ] "chat://{{top-level-header}}/{{id}}"
        - [ ] seach in file below system prompt
        - [ ] print to tool response
    - [ ] "https://path/to/webpage"
        - [ ] fetch webpage
        - [ ] parse into LLM-readable paragraphs
        - [ ] print to tool response
    - [ ] freeform text
        - [ ] in file search that has the same headers
        - [ ] on local search fail suggest LLM to use search_web instead
- [ ] implement `* Goal`
    - [ ] each subgoal should have a verification script that returns \
          0(shell) or \
          t(emacs-lisp) or \
          True(python) on success
    - [ ] passed verification should toggle todo list under `* Goal`
    - [ ] there should be `on_pass` and `on_fail` hook script too
- [ ] implement subagents
    - idea: `emacs --batch --visit abc.cyb.org ...` \
            reads `./trigger.org` and \
            outputs `./artifact.org`(overrideable), \
            which can tangle into expected file(s) specified in trigger.org
    - [ ] implement `builder.cyb.org`
        - idea: builds a new cyborg; artifact.org can be renamed into .cyb.org to work
    - [ ] make bash calling `builder.cyb.org` a tool
    - [ ] make calling the made agent a tool
    - maybe: `call_agent(agent_name, prompt)` tool \
             will try to trigger `../agent_name.cyb.org`?
