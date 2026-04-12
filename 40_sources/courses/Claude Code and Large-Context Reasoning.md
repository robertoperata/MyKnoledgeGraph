---
tags:
  - claude
  - genai
feature:
type:
author: "[[Tim Warner]]"
source: https://learning.oreilly.com/live-events/claude-code-and-large-context-reasoning/0642572265359/
---
# Claude Code and Large-Context Reasoning
https://github.com/timothywarner-org/claude-code

## Foundations
when create a new prompt we send all the context window (available data - token length) that is tokenised (convert to an atomic format that llm can understand natively= vector embedding).
When the context reach the limit of the available data it starts to evict or eject data that it doesn't feel relevant anymore to make room for new incoming.
this is NOT FIFO (the oldest get evicted). These LLMs in general hold on the initial suff and the most recent stuff most tightly. And the conversation in-between is most likely subject to getting evicted.
context is:
- any prompt 
- custom instruction (.claude.md?)
- vendor system message (responses?`)
## MCP

## Agents

## Skills
