  > analyze how the qwen code subagent system works. and how a prompt like following trigger the actions within subagents.
      Use prompts like: "Use an agent to explore the Terraform modules in /infra and another agent to analyze the
    wrangler.toml configs, then synthesize findings" or simply let Qwen auto-invoke agents when it's helpful

  ✦ The user wants to understand how the subagent system works in detail, specifically:

     1. How the subagent system is architected
     2. How a prompt like "Use an agent to explore X and another agent to analyze Y, then synthesize findings" triggers
        the actions within subagents
     3. How Qwen auto-invokes agents when it's helpful

    Let me study the relevant code files to give a comprehensive answer.

  ✦ Let me study the subagent system in detail.

  ╭──────────────────────────────────────────────────────────────────────────────────────────────────────────────────╮
  │ ✓  Agent Explore subagent system architecture                                                                    │
  │                                                                                                                  │
  │     ✔ Explore: Explore subagent system architecture · 31 tools · 1m 14s · 379k tokens                            │
  │                                                                                                                  │
  │ ✓  Agent Explore prompt injection for agents                                                                     │
  │                                                                                                                  │
  │     ✔ Explore: Explore prompt injection for agents · 50 tools · 1m 34s · 906k tokens                             │
  ╰──────────────────────────────────────────────────────────────────────────────────────────────────────────────────╯

  ✦ Excellent. I now have a comprehensive understanding of the subagent system. Let me synthesize this into a clear,
    structured explanation that answers the user's question about:

     1. How the subagent system works architecturally
     2. How a prompt like "Use an agent to explore X and another agent to analyze Y, then synthesize findings" triggers
        actions within subagents
     3. How Qwen auto-invokes agents when helpful

    Let me write a thorough but clear response.

  ✦ Here's a deep-dive into how the Qwen Code subagent system works and how prompts trigger it.

    Architecture: Five Layers

      1 LLM (model)
      2   │  calls `agent(...)` function (JSON function-call)
      3   ▼
      4 AgentTool (agent.ts)  ─── resolves subagent config, approval mode, fork vs. named
      5   ▼
      6 AgentHeadless (agent-headless.ts)  ─── lifecycle wrapper (start → run → finish)
      7   ▼
      8 AgentCore (agent-core.ts)  ─── inner reasoning loop (LLM → tools → LLM → ...)
      9   ▼
     10 CoreToolScheduler + GeminiChat  ─── actual tool execution + API calls

    How the LLM "Knows" About Agents

    Three mechanisms:

    1. Core system prompt (client.ts → prompts.ts)
    The main instruction tells the LLM:

     │ Subagent Delegation: When doing file search, prefer to use the agent tool to reduce context usage. Proactively
       use the agent tool with specialized agents when the task matches the agent's description.

    2. Dynamic function declaration (agent.ts constructor)
    The agent tool's JSON schema is rebuilt every time subagents change. The LLM sees something like:

      1 Tool: agent
      2   Parameters:
      3     - description (string, required): 3-5 word summary
      4     - prompt (string, required): Task to perform
      5     - subagent_type (enum): [Explore, general-purpose, statusline-setup, ...]
      6     - run_in_background (boolean, optional)
      7
      8 Description: Launch a new agent to handle complex, multi-step tasks...
      9 Available agent types:
     10   - **Explore**: Fast codebase exploration (read-only, uses 'fast' model)
     11   - **general-purpose**: Research complex questions, search code, multi-step tasks
     12   - **statusline-setup**: Configure status line setting
     13 Launch multiple agents concurrently whenever possible...

    The subagent_type parameter has an enum constrained to all available agents — the LLM can only choose from the
    list.

    3. Runtime system reminder (client.ts, every turn)
    If non-built-in subagents exist, a <system-reminder> tag is injected:

     │ You have powerful specialized agents at your disposal, available agent types are: [custom names]. PROACTIVELY use
        the agent tool to delegate user's task to appropriate agent when user's task matches agent capabilities.

    This reminder is injected into every turn of every user query.

    What Happens When the LLM Says "Use an agent..."

    Explicit Prompt Example

    When you say: "Use an agent to explore the Terraform modules in /infra and another agent to analyze the
    wrangler.toml configs, then synthesize findings"

    Step 1 — LLM parses intent and emits parallel function calls

    The LLM's function-calling layer produces a single message with two agent tool calls:

      1 // Call 1
      2 { "name": "agent", "args": {
      3     "description": "Explore Terraform modules",
      4     "prompt": "Explore all Terraform modules in /infra...",
      5     "subagent_type": "Explore",
      6     "run_in_background": false
      7 }}
      8
      9 // Call 2
     10 { "name": "agent", "args": {
     11     "description": "Analyze wrangler.toml configs",
     12     "prompt": "Read and analyze all wrangler.toml files...",
     13     "subagent_type": "Explore",
     14     "run_in_background": false
     15 }}

    Both calls arrive in the same ToolCallRequest event from the LLM stream.

    Step 2 — CoreToolScheduler partitions them

    In coreToolScheduler.ts, partitionToolCalls() groups calls into batches. Agent tool calls are always
    concurrency-safe (isConcurrencySafe returns true) — they spawn independent subprocesses with no shared state. Both
     calls go into a single concurrent batch.

    Step 3 — Parallel execution

    The scheduler runs both agents in parallel (configurable concurrency cap, default 10):

      1 Agent 1 (Explore): runs in its own AgentCore loop
      2   → createChat() → GeminiChat with "fast" model
      3   → Tools: read_file, grep, glob, shell, ls, web_fetch, ... (read-only)
      4   → Executes: glob **/*.tf, read_file infra/modules/*, etc.
      5   → Returns final text: "Found 3 Terraform modules: ..."
      6
      7 Agent 2 (Explore): runs in its own AgentCore loop
      8   → Same setup, different prompt
      9   → Executes: glob **/wrangler.toml, read_file etc.
     10   → Returns final text: "Found 2 wrangler.toml files with configs: ..."

    Each agent uses the Explore built-in agent type, which:
     - Uses a "fast" model (cheaper/quicker)
     - Has a read-only tool allowlist (no write/edit/delete)
     - Has a system prompt that prohibits file creation

    Step 4 — Results injected as function responses

    Both agents complete. Their outputs become function-response parts in the chat history:

     1 [FunctionResponse: agent#1] → "Found 3 Terraform modules: ..."
     2 [FunctionResponse: agent#2] → "Found 2 wrangler.toml configs: ..."

    Step 5 — Parent LLM synthesizes

    The parent LLM's next turn receives both results. It has no built-in aggregation engine — the LLM itself is the
    synthesis layer. It reads both responses and generates a synthesized answer to the user.

    Background Mode Alternative

    If run_in_background: true is set (either explicitly or because the agent definition has background: true):

     1. The agent is fire-and-forget: returns an immediate placeholder "Background agent launched — agentId: abc123"
     2. The parent LLM continues its loop without waiting
     3. When the background agent finishes, a notification is delivered via SendMessage
     4. The parent LLM picks up the result in a subsequent turn

    Auto-Invocation: How Qwen Decides to Use Agents Proactively

    The system prompt tells the LLM to proactively use agents without being asked. The heuristic the LLM is trained
    (via prompting) to follow:

     1. For file search → prefer agent (especially Explore) over doing it inline, to save context tokens
     2. When the task matches an agent's description → delegate
     3. For simple operations (reading a specific file, grepping a keyword) → the prompt explicitly says NOT to use
        agents for these
     4. For complex, multi-step tasks → delegate to general-purpose agent

    The agent tool's description also gives explicit guidance about when not to use it:

     │ When NOT to use the Agent tool:
     │ - If you want to read a specific file path → use read_file
     │ - If you are searching for a specific class definition → use grep_search
     │ - Other tasks that are not related to agent descriptions

    So the decision to auto-invoke is entirely prompt-driven — the LLM's reasoning decides when delegation saves
    context vs. when a direct tool call is simpler.

    Agent Isolation

    Each subagent gets full isolation:

     - Config override: Object.create(parentConfig) + rebuildToolRegistryOnOverride() — isolated FileReadCache, core
       tools bound to subagent config
     - Tool filter: EXCLUDED_TOOLS_FOR_SUBAGENTS — blocks agent (prevents recursive spawning), cron_*, task_stop,
       send_message
     - Model isolation: Subagent can use a different model/provider — parseSubagentModelSelection() resolves fast,
       inherit, or a specific authType:modelId
     - Approval mode isolation: Parent's yolo/auto-edit propagates; otherwise subagent's own approval mode applies

    Key Files Summary


    ┌──────────────────────────────────┬───────────────────────────────────────────────────────────────────┐
    │ File                             │ What It Does                                                      │
    ├──────────────────────────────────┼───────────────────────────────────────────────────────────────────┤
    │ agent.ts (1746 lines)            │ The agent tool: dynamic schema, execution dispatch, fork creation │
    ├──────────────────────────────────┼───────────────────────────────────────────────────────────────────┤
    │ agent-core.ts (1679 lines)       │ The inner reasoning loop for all subagents                        │
    ├──────────────────────────────────┼───────────────────────────────────────────────────────────────────┤
    │ agent-headless.ts                │ Lifecycle wrapper: start → run → finish events                    │
    ├──────────────────────────────────┼───────────────────────────────────────────────────────────────────┤
    │ subagent-manager.ts (1224 lines) │ CRUD, caching, priority-chain resolution, createAgentHeadless()   │
    ├──────────────────────────────────┼───────────────────────────────────────────────────────────────────┤
    │ builtin-agents.ts                │ Explore, general-purpose, statusline-setup definitions            │
    ├──────────────────────────────────┼───────────────────────────────────────────────────────────────────┤
    │ coreToolScheduler.ts             │ Batch partitioning, concurrency, routing tool calls               │
    ├──────────────────────────────────┼───────────────────────────────────────────────────────────────────┤
    │ client.ts (line 1218)            │ System reminder injection each turn                               │
    ├──────────────────────────────────┼───────────────────────────────────────────────────────────────────┤
    │ prompts.ts (line 333)            │ Core subagent delegation instruction                              │
    ├──────────────────────────────────┼───────────────────────────────────────────────────────────────────┤
    │ fork-subagent.ts                 │ Fork mechanism (cache-parity subagents)                           │
    └──────────────────────────────────┴───────────────────────────────────────────────────────────────────┘
