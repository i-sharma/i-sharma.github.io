Discarded:
- 2025-03-13-mcp.md


# 2024-07-22-finding-chords.md

---

# 2025-03-13-react-codeact.md

---

# 2025-03-13-writing-prompts.md

### How to prompt

- explain briefly and to the point what needs to be done. be very specific
- provide enough context
- Sample output
- roleplay
- guidelines
  

How to test prompts and tool descriptions?
  We put a lot of effort into the descriptions and specs for these tools across a wide variety of agentic tasks. We tested them to uncover any ways that the model might misunderstand the spec, or the possible pitfalls of using the tools, then edited the descriptions to preempt these problems. We believe that much more attention should go into designing tool interfaces for models, in the same way that a large amount of attention goes into designing tool interfaces for humans.
  https://www.anthropic.com/engineering/swe-bench-sonnet

  https://platform.openai.com/docs/guides/prompt-generation

- Context Window
- Oneshot 
- Roleplay
- Tell it specifically what to do
 
---

# 'Notes on ai agents - from notion.md'

```markdown
[Converging on a General-Purpose AI Agent Architecture.pdf](attachment:81f5a03a-4aa3-4c96-a89e-32d5945ad7f4:Converging_on_a_General-Purpose_AI_Agent_Architecture.pdf)

**One Agent**  

- CodeAct agent loop
- context management (need to research this: compression and manus file trick)
- knowledge management (through prompts or web search or RAG like tools)
- tools and tool interface
    - some contrained tools + arbitrary python code (codeact) - should experiment and write about clear guidelines - when to fix tool interface vs when to allow arbitrary code execution
    - computer use tools
- system prompt
    - Very imp knob - You should take inspiration from great agents like manus, claude code, etc.
    - [9 Lessons From Cursor's System Prompt](https://byteatatime.dev/posts/cursor-prompt-analysis/)
    - https://docs.anthropic.com/en/release-notes/system-prompts
    - 
    
    [Claude code system prompt - 2](https://www.notion.so/Claude-code-system-prompt-2-24882cad0c73801484c3f6674282574b?pvs=21)
    
    [Claude code system prompt - 1](https://www.notion.so/Claude-code-system-prompt-1-24882cad0c738069876af5ef8d91bad5?pvs=21)
    
    [Manus - Technical report](https://www.notion.so/Manus-Technical-report-24882cad0c7380d894f6d3111b634e50?pvs=21)
    
    [Manus - tools and prompts](https://www.notion.so/Manus-tools-and-prompts-24882cad0c7380e588a6e6f7baea050d?pvs=21)
    
- agent memory management
- sandbox (e2b like solutions don’t exist for windows it seems)
- dynamic model selection based on task
- Context engineering
    - https://x.com/LangChainAI/status/1950226846538485918?referrer=grok-com
    - https://manus.im/blog/Context-Engineering-for-AI-Agents-Lessons-from-Building-Manus?ref=blog.langchain.com

**Multi Agent**

- One orchestrator + planner
- sug agents - experts in different areas
    - prevents context pollution
- You should run single vs multiple agents experiments - to determine efficacy and cost of multi agent setup

- react agent - anthropic blog post on building effective agents
- Langgraph implements many agent architectures - agents as graphs
    - learning langgraph through its docs: [1. Build a basic chatbot](https://langchain-ai.github.io/langgraph/tutorials/get-started/1-build-basic-chatbot/)
- Langgraph deepagents - https://www.youtube.com/watch?v=TTMYJAw5tiA
    - 4 components make up a deep agent
    - planning tool/todolist tool
    - sub agents
    - system prompt
    - file system (save lengthy observations, etc.) somewhere in the file system and store reference to that file in the actual context of the agent
- ? Writing system prompt and tool description
- ? Agent memory is state?
- ? Is there a public implementation of claude code?
- ? Memory and MCP

# Resources

- https://www.perplexity.ai/search/has-the-community-converged-on-ticj8fxtRtSPfvXAPig5rQ
- https://manus.im/blog/Context-Engineering-for-AI-Agents-Lessons-from-Building-Manus?ref=blog.langchain.com
- https://chatgpt.com/share/689240d5-4ca4-8004-b19f-02ef4efaefcc
- https://gist.github.com/jlia0/db0a9695b3ca7609c9b1a08dcbf872c9
- https://www.anthropic.com/engineering/built-multi-agent-research-system
- https://gist.github.com/renschni/4fbc70b31bad8dd57f3370239dccd58f
- https://byteatatime.dev/posts/cursor-prompt-analysis/
- https://docs.anthropic.com/en/release-notes/system-prompts
```


---

# RAII-in-python.md

- RAII in cpp
- with - __enter__, __exit__
- async with - __aenter__, __aexit__

---

# 'all tech is tools.md'

---

# 'claude code as a general purpose agent.md'

claude code + mcps

experiments
https://docs.google.com/document/d/1x7C4xnsCGyevlFSmADgt-4D8lLQidByXWBA3imAw0gc/edit?tab=t.0

gemini cli mcps experiments as well

---

# 'llm temperature.md'

tweet - see gpt
https://chatgpt.com/c/68a0e085-3780-8326-ba45-0cba09369be1

---

# most-imp.md

let's address this issue:
https://github.com/google-gemini/gemini-cli/issues/6219
fetch the issue, understand it, then walk me through your plan to fix it.
we'll need to clone the git repo when I approve your plan

with claude code, gemini, cline, augment code and openhands

+ build your own agent bench 

then take things from deepagents and manus to test them out


Fucking awesome - like that building browser blog that's been getting attention

---

# on_experimental_workbench.md

Ishan Repost Tweet

---

# on_modes_of_working.md

- Execution Mode
- Constraint Goal-oriented thinking
- Free-mode
- Here you are free to explore: let your mind wonder, don't force an outcome

Your mind takes what it has learnt in Execution modes + your goal and shows your new paths.

---

# playwright-mcp.md

- what is mcp
- setup mcp and python client (without llm) test
  - transports
  - important notes about the mcp protocol
- playwright mcp

---

# trio.md

https://vorpus.org/blog/notes-on-structured-concurrency-or-go-statement-considered-harmful/#go-statement-considered-harmful

anyio

---