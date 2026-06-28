- [X] implement RAG
  - [X] refactor `* System Prompt`
  - [X] refactor how we read `* System Prompt`(read only the md block)
  - [X] add wiki section
  - [X] create a fetch tool that can handle:
    - [X] "wiki://path/to/concept"
    - [X] "chat://{{top-level-header}}/{{id}}"
    - [X] "https://path/to/webpage"
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
