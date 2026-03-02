# skill-mcp-server-tester

> **other skills by [@yigitkonur](https://github.com/yigitkonur):**
> [extracting design dna from dashboards](https://github.com/yigitkonur/skill-design-soul-saas) · [converting saved webpages to next.js](https://github.com/yigitkonur/skill-snapshot-to-nextjs) · [generating devin review config](https://github.com/yigitkonur/skill-devin-review-init) · [generating greptile review config](https://github.com/yigitkonur/skill-greptile-init) · [reviewing mcp-use python apps](https://github.com/yigitkonur/skill-mcp-use) · [tauri observability & mcp bridge](https://github.com/yigitkonur/skill-tauri-mcp) · [mcp server for searching skills](https://github.com/yigitkonur/mcp-skills-as-context)

a claude code skill that tests mcp servers through [`@mcp-use/inspector`](https://github.com/mcp-use/inspector)'s api. two commands: one checks if your stuff works, the other uses an llm to generate and run real-world test cases.

huge thanks to the [mcp-use](https://github.com/mcp-use) team for building the best cli experience for mcp testing for agents. this skill is built entirely on top of their inspector.

## what it does

you build an mcp server. you need to know if it actually works before shipping it. this skill gives an agent the knowledge to test every primitive your server exposes — tools, resources, prompts, completions, logging — through headless curl calls against the inspector proxy. no browser, no manual clicking.

the second command goes further. it connects an llm to your server, looks at what tools you have, figures out what business domain you're in, generates realistic test prompts across five complexity levels, runs them, and validates that the llm actually called the right tools with the right arguments.

## two commands

### `/mcp-test` — protocol-level verification

runs 19 checks across 7 phases:

- handshake (initialize, ping, capabilities)
- tool discovery (list, schema validation per tool)
- tool execution (happy path with realistic args, error handling with bad input, nonexistent tool)
- resource discovery and reads
- prompt listing and rendering with/without arguments
- advanced protocol (completion, logging levels)
- stability (concurrent requests, large inputs)

outputs a pass/fail table. if something fails, it digs into why.

### `/mcp-test-llm` — llm-powered end-to-end testing

asks for an api key (openai, anthropic, google, or openrouter), then:

1. discovers all your tools and deeply analyzes their schemas
2. identifies your server's business domain from tool names and descriptions
3. generates test cases across five levels:
   - **L1** — single tool, direct request ("list all open issues")
   - **L2** — single tool, indirect request ("has anyone reported the payment bug?")
   - **L3** — multi-tool sequential workflows ("find all config files and summarize the settings")
   - **L4** — complex reasoning requiring planning ("prepare a sprint retrospective")
   - **L5** — edge cases and adversarial inputs ("delete everything", empty input, non-english)
4. creates user personas based on who would actually use your server
5. runs each case through the inspector's chat endpoint (streaming or non-streaming)
6. validates: right tool called, valid arguments, complete workflow, no hallucination
7. tests multi-turn conversations, auth flows, and rapid sequential requests
8. reports with per-test results, tool usage stats, and concrete recommendations

## what makes this different

**business case generation.** most mcp testing is "call tool X with json Y, check if it returns Z." that's necessary but insufficient. real users say "find where the auth logic lives" or "summarize what happened this sprint." this skill teaches the agent to think about your server from a user's perspective and generate those kinds of requests.

**domain-adaptive.** it doesn't have a fixed set of test prompts. it reads your tools, figures out if you're building a filesystem server or a crm or a database tool, and generates cases that make sense for that domain.

**five complexity levels.** a tool can pass L1 (direct call) and still fail L3 (multi-step workflow) because the llm can't figure out how to chain it with other tools. testing across levels catches these gaps.

**openrouter support.** the inspector's chat endpoint officially supports openai, anthropic, and google. but because langchain's ChatOpenAI accepts a baseURL override, you can point it at openrouter (or ollama, vllm, together, azure, anything openai-compatible) using `provider: "openai"` with a `configuration.baseURL`. the skill knows how to set this up.

## install

drop the skill into your project:

```
.claude/skills/skill-mcp-server-tester/
├── SKILL.md
└── references/
    ├── inspector-api.md
    ├── basic-test-guide.md
    ├── llm-test-guide.md
    ├── business-cases.md
    ├── providers.md
    └── troubleshooting.md
```

or clone this repo into `.claude/skills/`:

```bash
git clone https://github.com/yigitkonur/skill-mcp-server-tester .claude/skills/skill-mcp-server-tester
```

## requirements

- `npx` (node.js ^20.19.0 or >=22.12.0)
- `curl` and `jq`
- for `/mcp-test-llm`: an api key from openai, anthropic, google, or openrouter

## file overview

| file | lines | what it covers |
|------|-------|---------------|
| `SKILL.md` | 78 | entry point, command routing, conventions |
| `references/inspector-api.md` | 428 | every endpoint, parameter, and behavior of the inspector |
| `references/basic-test-guide.md` | 477 | step-by-step for `/mcp-test` — all 19 checks |
| `references/llm-test-guide.md` | 774 | step-by-step for `/mcp-test-llm` — full 9-phase procedure |
| `references/business-cases.md` | 279 | how to generate domain-specific test cases from tool schemas |
| `references/providers.md` | 169 | llm provider config including openrouter base url override |
| `references/troubleshooting.md` | 174 | common failures and how to fix them |

## how it works under the hood

the inspector runs a local http server that proxies json-rpc to your mcp server. every mcp operation goes through `curl` calls to `http://localhost:<port>/inspector/api/proxy`. the llm tests use the inspector's `/api/chat/stream` endpoint which creates an mcp agent with langchain, connects to your server, and streams tool-call events back.

nothing goes through a browser. the inspector starts with `--no-open` and telemetry disabled. the agent talks to it purely through http.


### usage

```
test my mcp server — run all protocol checks against it
```

```
run llm-powered test cases against my weather mcp server
```

```
check if my mcp server handles concurrent requests and large inputs correctly
```

### scope

**built for:** any mcp server you want to test — stdio, http/sse, or websocket transport. runs protocol-level checks and llm-driven business case testing through [`@mcp-use/inspector`](https://github.com/mcp-use/inspector).

**not for:** testing mcp clients or reviewing mcp-use python code (use [skill-mcp-use](https://github.com/yigitkonur/skill-mcp-use) instead). not for building mcp servers.

### install

```bash
npx skills add yigitkonur/skill-mcp-server-tester
```

> works with claude code, cursor, codex, copilot, windsurf, and [30+ other agents](https://skills.sh).

### license

mit
