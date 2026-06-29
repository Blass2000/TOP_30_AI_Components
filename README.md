# TOP_30_AI_Components

The Core Building Blocks of AI Agents- I wanted to baseline and keep a list of the top AI components when designing an AI solution at an Enterprise level. 

## 1. Agent
The word agent is everywhere now. Every new AI tool wants to call itself an agent. But because of that, the meaning has become a little blurry. So let’s make it simple. An AI agent is usually an LLM that does not just answer once and stop. It runs in a loop. It can understand a goal, decide the next step, use tools, read the result, and then decide what to do next again. That loop is the important part. A normal chatbot works like this:

You ask a question → it gives an answer.

An agent works more like this:
You give a goal → it thinks about the next step → uses a tool → checks the result → continues until the task is done.
Press enter or click to view image in full size
So instead of producing one final answer, the agent produces a chain of actions.
Each action depends on what happened before. Coding is one of the clearest examples. You can ask an agent to debug a failing test. It may inspect the error, open the related file, change some code, run the test again, see another error, fix that, and continue until the test passes.
That is where agents are useful. They are helpful when the task is not fully predictable from the start.
For example:

“Debug this failing test.”
“Research this topic and summarize the best sources.”
“Check these support tickets and draft replies.”
“Review this codebase and find the issue.”
In all these cases, the next step depends on the previous result. That is when an agent makes sense. But you do not need an agent for everything. If the task is simple, use a simple solution. If you only need to format a date, convert JSON, rename a file, or generate a short answer, a normal prompt or a small script is better. Because agents are not free. Every loop costs time. Every tool call costs money.

And the longer the loop becomes, the harder it is to predict what the agent will do. Debugging also becomes harder because the agent may not always make the same choices in the same order.

So the rule is simple:

Use a normal prompt for simple answers. Use a script for fixed steps. Use an agent when the task needs flexibility, decisions, and feedback from each step. The goal is not to use agents everywhere. The goal is to use them where their flexibility is actually worth the cost.

## 2. Execution Model
An agent loop usually follows a simple pattern. It is not magic. It is just a repeated cycle of three steps:
Think → Act → Observe

First, the model thinks. It reads the current conversation, looks at the goal, checks the available context, and decides what should happen next. Then the model acts. This usually means it calls a tool.

That tool can be anything the system gives access to: reading a file, running a command, searching a database, calling an API, using MCP, or asking another service for help.

But the model does not directly run everything by itself. There is usually a layer around the model that receives the tool call, checks whether it is valid, runs it safely, and then returns the result. You can think of this layer as the “controller” around the agent. Finally, the model observes.

The result from the tool comes back and becomes part of the conversation. Now the agent has new information. So it starts the next round with this updated context.
That is the loop:

Think → Act → Observe → Think again

<img width="700" height="425" alt="image" src="https://github.com/user-attachments/assets/ba602f20-2563-4f94-bc87-05161bb2ad1b" />

This pattern has different names. Some people call it ReAct. Some call it Think-Act-Observe. Some simply call it an agent loop. The name is different, but the idea is the same. The model does not try to predict the whole path in one shot. It takes one step, checks what actually happened, and then decides the next step based on the real result. That is what makes agents useful.

A normal LLM call has to answer based on what it already knows in that moment. But an agent can keep learning from the task while doing it. For example, imagine an agent is fixing a failing test. It runs the test. The test fails. The error message comes back. Now the agent can read the stack trace, open the related file, make a change, and run the test again.

If the next error is different, that becomes the next observation. The agent does not need to get everything right on the first try. The loop gives it a chance to recover. This is also why agents can feel more powerful than normal prompts. They can make mistakes, see the result, and correct themselves in the next step.

But there are two important variations to understand. The first one is parallel tool calls. Sometimes an agent does not call only one tool at a time. It may call multiple tools together. For example, it may read three files at once instead of reading them one by one. This can save time, especially in research or codebase analysis tasks.

But it can also create problems. If two tool calls try to edit the same file or change the same thing, conflicts can happen. So parallelism is useful, but it needs control. The second one is blocking vs non-blocking execution.

Most agents work in a blocking way. That means the agent calls a tool, waits for the result, and then continues. Simple. But some agents can run jobs in the background. For example, they may start a long-running task and continue doing something else while waiting.

This is called non-blocking or asynchronous execution. It can be powerful for bigger workflows, but it also makes the system harder to manage. So at the beginner level, just remember this: An agent works by repeating a loop. It thinks about the next step. It acts by using a tool. It observes the result. Then it repeats until the task is complete. That loop is the heart of agentic engineering.
## 3. Agent State
In agentic engineering, the word state can mean two different things. The first meaning is about workflow progress. For example:

Where is the agent right now? Which step has it completed? What still needs to happen? That kind of state is about tracking the task. But here, we are talking about the second meaning:

What does the agent know at this moment?

That is the agent’s state. And it usually has two parts. The first part is the context window. This is everything the model can see right now. It includes your latest message, the system instructions, previous tool calls, tool results, and any other information that has been added to the current conversation. You can think of it like the agent’s current working memory.

<img width="1100" height="619" alt="image" src="https://github.com/user-attachments/assets/d5190014-b8c1-43bf-b38d-4803b2a28c04" />
But it has limits. The model can only hold a fixed amount of text at once. That limit is called the token limit or context limit. And even before the hard limit is reached, the context can become messy. Too much old information can make the agent less focused.

Also, when the session ends, this context usually disappears. The second part is everything outside the context window. This includes things the model cannot see unless it fetches them.

For example:

Files on disk
Database records
Saved memory
API results
Search results
Documentation
Project history
The model does not automatically know all of this. It cannot reason over a file if the file was never opened. It cannot use a database record if it was never fetched. It cannot remember a past decision unless that memory is brought back into the current context.

So the agent only works with what is visible to it right now . Everything else must be pulled in when needed. This is an important idea. An agent may have access to many tools and data sources, but access is not the same as awareness. If the information is not in context, the model is not truly using it yet.

So where should agent state live?

For most developer workflows, files are the best default. Files are easy to read. Easy to edit. Easy to track with Git. Easy to compare through diffs. And both humans and agents can work with them naturally. Use memory for facts that should survive across sessions but do not need a full Git history.

For example, a user preference, a project rule, or a repeated instruction. Use a database when the state needs structure. For example, when many users, agents, or processes need to query and update the same information. A database makes sense when you need filtering, searching, relationships, or shared access.

But state becomes harder when you use multiple agents. If two agents read the same file, that is usually fine. But if two agents write to the same file at the same time, problems can happen. One agent may overwrite the work of another. This is the classic race condition. That is why isolated workspaces are useful.

For coding agents, Git worktrees can help because each agent gets its own working copy. They can work separately and merge changes later. Subagents are a little easier to manage. A subagent usually starts with a fresh context window. The parent agent gives it only the information it needs for that specific task. This keeps the subagent focused.

But there is a simple warning sign:

If the parent has to pass a huge amount of context to the subagent, the task may not be split correctly. A good subagent task should be narrow. It should not need the whole world to do its job. So the simple way to think about agent state is this:

The context window is what the agent can see right now. Files, memory, and databases are where information can live outside the model.

And good agent design is mostly about deciding what should stay outside, what should come inside, and when.
## 4. Common Agent Patterns
Once you start using more than one agent, a new question appears:

How should these agents work together?

Because one agent can do a lot. But multiple agents can make a workflow cleaner, faster, and easier to control, if they are designed properly. There are a few common patterns that show up again and again.

The first one is the planner/executor pattern.
<img width="1100" height="619" alt="image" src="https://github.com/user-attachments/assets/44f47cbc-cc36-4d73-89fd-ce3379448123" />
In this pattern, one agent creates the plan, and another agent does the actual work. The planner thinks through the task. The executor follows the plan and takes action. This split is useful because planning and execution need different kinds of focus. Planning is open-ended. Execution is more direct.

For example, if you ask an AI system to build a feature, the planner may break the work into steps:

First update the database schema. Then add the API. Then update the frontend. Then write tests. After that, the executor can start working through those steps one by one. This pattern is useful for long tasks where you do not want the agent to jump straight into code without thinking first.

The second pattern is router/specialist.
<img width="1100" height="619" alt="image" src="https://github.com/user-attachments/assets/75d9a31b-9270-49c4-a380-7ec7118f9fb1" />

Here, one agent acts like a router. It reads the incoming request and decides which specialist agent should handle it. Each specialist is designed for a specific type of work.

For example:

A security reviewer
A debugging specialist
A documentation writer
A test writer
A code reviewer
This makes the system easier to manage. Instead of one big agent trying to do everything, each specialist has a narrower role, a clearer prompt, and a smaller set of tools. That usually makes the behavior more predictable. It can also be cheaper because not every task needs the biggest model or the most powerful agent.

The third pattern is map-reduce parallelism.
<img width="1100" height="619" alt="image" src="https://github.com/user-attachments/assets/e6fbc9d9-d8ac-4301-83fe-f92543e8e914" />

This sounds technical, but the idea is simple. You split one big task into many smaller tasks. Then multiple agents work on those smaller pieces at the same time. After that, another agent combines the results into one final output. For example, imagine you want an agent to review a large pull request.

Instead of giving the whole pull request to one agent, you can split it by file. One subagent reviews file one. Another reviews file two. Another reviews file three. Then an aggregator agent collects all the reviews and creates one final summary.

This is useful for read-heavy work like code review, research, document analysis, and large content reviews. It can save time because many parts run in parallel. But the final quality depends on how well the results are merged. If the aggregator misses important details, the final answer can still be weak.

These patterns are not separate boxes that you must choose from. Real agent workflows often combine them. A planner may create the task plan. A router may send different parts to specialist agents. Those specialists may work in parallel.

Then another agent may merge the results and send them back for final review. The important part is the handoff. Every time one agent passes work to another agent, it needs to pass the right amount of context. Not too little. Not too much.

If the handoff is too small, the next agent may not understand the task. If the handoff is too large, the next agent may get confused or waste context. Good agent design is mostly about clean boundaries.

Where does one agent’s job end? Where does the next agent’s job begin? What information must be passed forward?

Those boundaries are where multi-agent systems usually succeed or fail. So the simple takeaway is this:

Use planner/executor when you want better planning before action.
Use router/specialist when different tasks need different experts.
Use map-reduce when a large task can be split into smaller pieces.
And no matter which pattern you use, make the handoff between agents clear. That is what keeps the whole system understandable.

## 5. Agent Config Files
Every agent starts with instructions. Before it answers, before it uses tools, and before it touches your code, there is usually a system prompt behind it. That system prompt tells the agent how the tool works, what format to follow, how to call tools, and how to behave inside that specific agent environment.

<img width="1100" height="733" alt="image" src="https://github.com/user-attachments/assets/0dbcb048-d9d1-4938-8eaf-03538c569cc0" />
But there is one problem. The default system prompt does not know your project. It does not know your coding style. It does not know your package manager. It does not know your folder structure. It does not know your team rules. So if you do not give the agent project-specific instructions, it will guess. And that is where problems start.

It may use npm when your project uses pnpm. It may suggest pip install when your Python project uses uv. It may format code with one tool when your project uses another. It may write defensive, overcomplicated code because that pattern appeared often in its training data.

This is why agent config files matter. An agent config file is a project-level instruction file. The agent loads it at the start of a session and keeps it in context while working. You can think of it as a rulebook for your project.

It tells the agent:

How this project works. Which tools to use. Which patterns to follow. Which things to avoid. What rules must never be broken. Claude Code uses a file called CLAUDE.md. Many other tools use AGENTS.md. Different names, same basic idea.

The goal is simple:

Before the agent writes even one line of code, it should read the rules of your project. A useful config file does not need to be long. In fact, shorter is usually better. A good agent config file may include things like:

The package manager your project uses
The test command
The lint command
Important folder conventions
Function length limits
Naming rules
Security rules like “never commit secrets”
Behavior rules like “always read a file before editing it”
These small instructions can save a lot of bad output. Without a config file, the agent follows whatever looks most likely. With a config file, the agent follows your project rules. But there is one mistake people make. They put too much into the config file. They copy a long AI-generated rules document. They add generic advice. They write things like “write clean code” or “use best practices.” That sounds useful, but it usually does not help much. The model already knows generic advice. What it needs is specific project guidance.

So keep your config file short!!!!!! please!! , sharp, and practical. Try to keep it under 100 lines. Remove anything that does not improve the agent’s work. Do not treat it like normal documentation. Treat it more like code. Review it when it changes. Improve it when the agent makes repeated mistakes.

Delete rules that are not useful anymore. A good config file is not there to impress the agent. It is there to reduce guessing. And that is the real value. The less your agent has to guess, the better it can work.
## 6. Reusable Workflow Files
Config files are always active. Reusable workflow files are different. They are loaded only when the agent needs them. You can think of them like small instruction guides for specific tasks.

For example:

One workflow file can explain how to write tests. Another can explain how to review a pull request. Another can explain how to migrate a database. Another can explain how to update documentation. The agent does not need all of these instructions all the time. It only needs the right one at the right moment. That is where reusable workflow files help. They are usually written in Markdown, but they also include a small metadata section at the top. This metadata is called YAML frontmatter.

It may include things like:

The name of the workflow
A short description
When the agent should use it
Which files or folders it applies to
For example, Claude Code has skills inside .claude/skills/. Cursor has rules. Different tools use different names, but the idea is similar:

Give the agent reusable instructions for a specific kind of task. The most important part is the description. The description tells the agent when this workflow is useful. If the description is clear, the agent can pick the right workflow at the right time. If the description is vague, the agent may ignore it or use it in the wrong place. Some workflow files also use globs. A glob is just a file-matching pattern.

For example, you can tell the agent that a workflow applies only to *.test.ts files, or only to files inside a docs/ folder. That keeps the instruction more focused. But the real value is not in the file format. The real value is in the quality of the instructions. A short, clear workflow can help a small model perform better because it gives the model a better process to follow.

There is an interesting lesson from research here. In SkillsBench, researchers tested 86 tasks across 11 domains and gave models short written workflows for solving those tasks.

The result was surprising. Claude Haiku with human-written skills scored better than Claude Opus without those skills.
<img width="1100" height="733" alt="image" src="https://github.com/user-attachments/assets/8d0ec29e-5fd6-4b26-a095-b5a6e225ba46" />

In simple words:

A cheaper model with good instructions performed better than a stronger model without them. That is a powerful idea. It means instructions matter. Process matters. Good workflows matter. But there is also a warning. When researchers allowed the model to write its own skills, the improvement disappeared.

That makes sense. Generic AI-generated instructions often become noisy. They sound useful, but they do not give the model clear guidance. They add more text without adding more value. And when agents get too much weak context, performance can get worse. So reusable workflow files should not be long generic documents. They should be short, specific, and based on real work. A simple way to separate things is this:

Use config files for rules that are always true. Use workflow files for task-specific procedures. Use the live prompt for what is unique about the current request.

For example:

Your config file may say:
“Use pnpm for this project.”
A workflow file may say:
“When adding a new API route, update the route file, add validation, write tests, and update docs.”
Your live prompt may say:
“Add a new endpoint for exporting student submissions.”
Each layer has its own job. The config gives the agent project rules. The workflow gives the agent a repeatable process. The prompt gives the agent the current task. When these three work together, the agent has less guessing to do. And less guessing usually means better output.


