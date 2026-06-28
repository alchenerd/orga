Work In Progress

Currently just a chatbot

---

# OrgA - An (aspiring) agent who lives in an org-mode file

## Dependency

1. emacs
2. openrouter key

## How to use

0. (Recommended) Open orga.org inside any other editor, or with emacs but don't allow evaluation; check what is inside first
1. Open orga.org inside emacs, allow eval
2. suppliment openrouter key in the file, or creaete .env `OPENROUTER_API_KEY=<YOUR_KEY_HERE>`, or load key in your environment
3. type anything under heading `** Input`
4. press C-c C-v
5. AI responds, goto 3

## Try headless

`emacs --batch --visit orga.org --eval "(progn (setq org-confirm-babel-evaluate nil) (org-babel-goto-named-src-block \"orga__initialize\") (org-babel-execute-src-block) (ai-chat-cycle) (save-buffer))"`

<img width="1914" height="711" alt="西元2026年06月28日 (週日) 23時02分14秒 CST" src="https://github.com/user-attachments/assets/a7f2a471-2f8d-4a56-a11d-cc8198f54c7a" />
