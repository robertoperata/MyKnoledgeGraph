---
tags: 
feature: 
type: 
author: 
source: 
---
# AI Talks - Lada Kesseler Augmented Coding

## Anti-pattern: context rot
create a `knowledge.md` posting the good answers and start a new conversation. "Load the important things"

create ground rules `CLAUDE.md` user-level or project-level

## Anti-pattern: distracted agent
"Be proactive and flag issues before they become problems"

## Pattern: Focused Agend
smaller scope == better result
one thing at a time!

code_style.md
tdd process

## 8) Pattern: Knowledge Composition
## 9) Semantic Zoom
zoom in:
- tell me more about
- what do you mean by
- how do you write
- explain better
zoom out:
- make it shorter
- make it more succint
- describe architecture on the high level
- give me a TLDR of this
## 10) Pattern: noise cancellation
give instruction to fight the default LLM verbosity

force succinctness:
- in knowledge documents 
- in responses to you
delete mercilessly
- use git as savepoint
temporary throw away docs are helpful:
- todo.md
- user_login_feature.md
- scratchpad.md

## 11) Pattern: Knowledge checkpoint

save in markdown + git commit = Knowledge checkpoint
commit before implement the feature

1. save feature to project.md
2. commit
3. look at the last commit and implement the change
## 12) Pattern: Parallel Implementations

