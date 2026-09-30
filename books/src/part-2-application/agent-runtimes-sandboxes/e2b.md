# E2B

> Isolated sandboxes for AI agents to execute code/process data/run tools safely

| Field | Value |
|-------|-------|
| Group | Agent Runtimes & Sandboxes |
| Type | API/SDK/Infra |
| Open Source | Yes |
| GitHub | [e2b-dev/E2B](https://github.com/e2b-dev/E2B) |
| Stars | 12186 |
| Documentation | [Official Docs](https://e2b.dev/docs) |

## Overview

E2B (Environment to Build) provides hosted, isolated Linux sandboxes designed for AI agents and LLM-driven code. The product gives an agent a fresh micro-VM with a file system, processes, internet access, and an interactive Python kernel, all behind a thin SDK. Use cases include code interpreters (the Sandbox surface is essentially "Code Interpreter as a service"), data-analysis agents, coding agents that need to compile and run code, and agentic workflows that produce or transform files. [1]

The product is open source (`e2b-dev/E2B`) and offered both as a managed cloud (default) and as self-hosted infrastructure. The execution layer uses Firecracker-style micro-VMs for strong isolation between sandboxes, so an agent running untrusted code cannot affect the host or other tenants. [1]

## Core Concepts

### Sandbox

A Sandbox is a running isolated environment with its own filesystem, processes, and network namespace. Each sandbox is created from a template (a pre-built filesystem image), boots in a few hundred milliseconds, and persists for the duration of its session — default 5 minutes of inactivity before automatic shutdown. Lifecycle is explicit: create → run code / interact with filesystem → `kill()` or auto-expire. [2]

### Templates

A Template is a snapshot of a base filesystem with pre-installed dependencies. E2B provides several stock templates (the default `code-interpreter` template ships Python with NumPy, Pandas, Matplotlib, scikit-learn, etc.). Users can build custom templates with the E2B CLI for project-specific environments (e.g., Node.js plus a specific package, or a Rust toolchain with a particular crate version). [3]

### Code Interpreter

The Code Interpreter is a higher-level wrapper around Sandbox that adds a Jupyter-style stateful Python kernel. `runCode()` returns structured output (text logs, errors, images for matplotlib plots, dataframes for pandas), making it easy for an LLM to consume execution results. [4]

### Persistence and Snapshots

A sandbox can be paused (snapshot taken) and later resumed with its full state (memory, filesystem, processes) intact. This is useful for long-running agent sessions that span LLM-call latencies, and for "auto-resume on request" patterns where a sandbox boots only when an incoming request needs it. [5]

### Internet Access

By default sandboxes have outbound internet access for fetching packages, calling APIs, downloading data. This can be restricted at the network policy level for higher-security tenants. [6]

## Architecture

A typical interaction:

1. Agent (running anywhere — local script, hosted service, serverless function) calls `Sandbox.create(template)` over HTTPS to E2B's control plane.
2. The control plane provisions a Firecracker micro-VM from the requested template; the VM boots in a few hundred ms.
3. Agent receives a sandbox handle and uses the SDK to:
   - Execute code (`runCode` for Code Interpreter; `commands.run` for arbitrary shell)
   - Manipulate the filesystem (`files.list`, `files.read`, `files.write`)
   - Tunnel inbound traffic (sandbox exposes ports; the SDK creates an HTTPS tunnel for the agent or browser to reach them)
4. Agent calls `sandbox.kill()` when done; or the sandbox auto-shuts down after the configured timeout.

Each sandbox is a fresh isolated environment; sandboxes do not share state by default. Persistence and snapshots are opt-in.

## Key Features and Functionality

### Code Execution

`runCode("python source")` executes a code block in the sandbox's Python kernel and returns:

- `logs.stdout` / `logs.stderr` — captured output
- `error` — exception details if the cell raised
- `results` — structured outputs from the last expression (rich types: text, png/jpg for plots, html for dataframes, latex, json)
- Execution is stateful across calls in the same sandbox session — variables defined in one call persist to the next. [4]

### Filesystem Access

The SDK exposes `files.list(path)`, `files.read(path)`, `files.write(path, content)`, and `files.download(path)`. An agent can drop a CSV into the sandbox, run analysis code on it, and read back the resulting plots or files. [4]

### Process Control

`commands.run(cmd)` runs shell commands; long-running processes return a handle that supports stdout streaming, signal sending, and explicit termination. PTY support is available for interactive shells. [7]

### Lifecycle Events and Webhooks

E2B exposes a Lifecycle Events API and webhooks so external systems can react to sandbox state transitions (start, pause, resume, kill) — useful for billing, observability, and orchestration. [5]

### Snapshots and Auto-Resume

A paused sandbox preserves memory, filesystem, and process state. Calling `connect(sandboxId)` resumes the exact prior state. The "auto-resume on request" pattern lets an idle sandbox stay snapshotted until inbound traffic arrives, then transparently resume. [5]

### Network Access

Sandboxes have outbound internet by default. Inbound traffic uses HTTPS tunnels: a sandbox can serve on port 8080, and the SDK returns a public URL that proxies to that port. This is the standard way to run a web app, Jupyter server, or API generated by the agent's code. [6]

### Storage Buckets

E2B can connect a sandbox to an external storage bucket (GCS, S3) so the agent can read and write large files without round-tripping through the SDK. [6]

### Git Integration

Native commands for cloning, committing, and pushing repositories from inside a sandbox — useful for coding agents that produce PRs. [5]

## Use Cases

### Code Interpreter

The canonical use case. Pair an LLM (Claude, GPT-5, etc.) with an E2B sandbox; the model writes Python, E2B runs it, the structured results go back into the prompt for the next turn. This is the architecture behind ChatGPT-style "advanced data analysis" features and many open-source equivalents.

### Data Analysis Agents

Agent receives a CSV/Parquet, uploads it to the sandbox, runs pandas/numpy code, returns plots and summary statistics. The structured `results` (matplotlib renders to PNG; dataframes to HTML) flow directly into the LLM context.

### Coding Agents

Agent writes Python/TS/Rust code, runs `pytest` / `npm test` / `cargo test` in the sandbox, observes output, fixes errors. Git integration enables full PR-producing workflows.

### Untrusted Code Execution

When users or LLMs supply code that may be malicious or buggy, run it in an E2B sandbox to contain side effects to the disposable VM.

### Agent Frameworks Integration

E2B documents first-class integrations with multiple agent frameworks: OpenAI Agents SDK, Anthropic, Vercel AI SDK, LangChain/LangGraph, plus partner agents like Amp and OpenClaw. [8]

## API Reference Summary

### Sandbox lifecycle

- `Sandbox.create(template?, options?)` — create new sandbox (defaults to `code-interpreter` template)
- `Sandbox.connect(sandboxId)` — reconnect to an existing/paused sandbox
- `sandbox.pause()` / `sandbox.kill()` — snapshot / destroy
- `Sandbox.list()` — list active sandboxes for the API key

### Code execution

- `sandbox.runCode(source, options?)` — execute Python in the stateful kernel
- `sandbox.commands.run(cmd, options?)` — execute arbitrary shell

### Filesystem

- `sandbox.files.list(path)`
- `sandbox.files.read(path)` / `sandbox.files.write(path, content)`
- `sandbox.files.download(path)` / `sandbox.files.upload(...)`

### Networking

- `sandbox.getHost(port)` — get public HTTPS URL forwarded to in-sandbox port

## Configuration

### Templates

Custom templates are defined with a Dockerfile-like `e2b.toml` plus a build script. Build with the E2B CLI; the resulting template is referenced by ID at `Sandbox.create(templateId)`. [3]

### Timeouts

Default sandbox timeout is 5 minutes of inactivity. Configurable per-create via `timeoutMs`; the maximum depends on plan tier.

### Environment Variables

Per-sandbox env vars are passed at create time or set with `sandbox.setEnvVars()`. [6]

## Integration Patterns

### With OpenAI Agents SDK / LangGraph / CrewAI / Pydantic AI

E2B publishes integration recipes for every major agent framework — the typical pattern wraps `runCode` as a tool the agent can call, with the sandbox lifecycle tied to the agent session. [8]

### With Claude (Anthropic)

Pair Claude tool-use with an E2B sandbox: the model emits a `tool_use` block with Python source; the runtime forwards it to `sandbox.runCode`; the structured result is returned as a `tool_result`.

### With Vercel AI SDK

Standard E2B+Vercel AI pattern: client-side React calls a server-side agent route which spawns an E2B sandbox per session and streams results back via Server-Sent Events.

### Self-Hosting

E2B is open source; production users with strict data-locality or compliance requirements can self-host the control plane and execution nodes. [1]

## Examples

### Minimal Python Quickstart

```python
from e2b_code_interpreter import Sandbox

sbx = Sandbox()                                # default template = code-interpreter
execution = sbx.run_code("print('hello world')")
print(execution.logs)
files = sbx.files.list("/")
print(files)
```

### Pandas Analysis from an LLM Tool Call

```python
sbx = Sandbox()
sbx.files.write("/data.csv", csv_bytes)

execution = sbx.run_code("""
import pandas as pd
df = pd.read_csv("/data.csv")
df.describe()
""")

# execution.results[0].html — rendered DataFrame, feedable to LLM
# execution.logs.stdout — print() output
```

### Long-Running Sandbox via Snapshot

```python
sbx = Sandbox(timeout=3600)
sid = sbx.sandbox_id

# ... time passes, agent goes idle ...
sbx.pause()

# later, in a different process:
sbx2 = Sandbox.connect(sid)
sbx2.run_code("print('resumed with prior state intact')")
```

## Limitations and Considerations

- **Per-sandbox cost**: Sandboxes consume metered compute; long-running unsnapshot sessions add up. Use timeouts and snapshots for idle periods.
- **Cold start vs warm**: Boot times are sub-second, but if your traffic pattern needs single-digit-ms response times, keep sandboxes warm or pool.
- **Memory limits per template**: Templates declare CPU/RAM limits; agents that exceed them get OOM-killed. Custom templates can request larger sizes (plan-gated).
- **Network policies in self-hosted setups**: Outbound internet access is on by default; in regulated environments you may need to lock down egress at the host firewall.
- **Stateful kernel persistence**: Variables persist across `runCode` calls within a sandbox session, but not across kill. Use snapshots if cross-session state matters.
- **Filesystem is ephemeral by default**: Files written inside the sandbox vanish on kill unless externally persisted (snapshot, storage bucket, or explicit download).

## Changelog Highlights

- **Snapshots & auto-resume**: Pause-and-resume with full state preservation
- **Storage bucket integration**: Direct GCS/S3 connection from inside the sandbox
- **Git integration**: First-class clone/commit/push from inside sandboxes
- **OpenAI Agents SDK integration**: First-class integration alongside existing LangChain/Anthropic/Vercel integrations
- **Lifecycle events API and webhooks**: External observability of sandbox state transitions [1][5][8]

## Citations

- [1] E2B Documentation - <https://e2b.dev/docs>
- [2] Running your first Sandbox - <https://e2b.dev/docs/quickstart>
- [3] Templates - <https://e2b.dev/docs/sandbox-template>
- [4] Code Interpreter SDK - <https://e2b.dev/docs/code-interpreting/analyze-data-with-ai>
- [5] Sandbox Lifecycle - <https://e2b.dev/docs/sandbox>
- [6] Internet Access and Networking - <https://e2b.dev/docs/sandbox/internet-access>
- [7] Commands and PTY - <https://e2b.dev/docs/sandbox/pty>
- [8] Agent Integrations - <https://e2b.dev/docs/agents/openai-agents-sdk>

## Discovery Signals

> Keywords, phrases, and user-intent patterns that should surface this chapter in semantic search.

### Keywords

- E2B
- code interpreter
- sandbox
- micro-VM
- Firecracker
- isolated execution
- code-interpreter template
- stateful Python kernel
- runCode
- commands.run
- filesystem access
- snapshot
- pause and resume
- auto-resume
- HTTPS tunnel
- inbound port forwarding
- storage bucket
- Git integration
- untrusted code execution
- agent runtime
- data analysis agent
- code-writing agent
- structured execution results
- PTY support

### Verb-Noun Tasks

- Spin up a Linux micro-VM for an AI agent in milliseconds
- Execute LLM-generated Python in a stateful Jupyter-style kernel
- Run pandas analysis on user-uploaded CSV files
- Capture matplotlib plots as PNGs back into the LLM context
- Read and write files inside an isolated sandbox
- Pause a sandbox and resume it later with full state preserved
- Build a custom sandbox template with the E2B CLI
- Expose a port from inside the sandbox via HTTPS tunnel
- Clone, commit, and push to git from inside the sandbox
- Sandbox untrusted code so it cannot affect the host
- Wrap `runCode` as a tool for OpenAI Agents SDK, LangChain, or Claude
- Connect a sandbox to an S3 or GCS bucket for large files

### User Intent Phrases

- How do I let an LLM run Python code safely?
- How can I build a ChatGPT-style data analysis feature?
- I need a sandboxed code interpreter for my coding agent
- How do I isolate user-supplied or LLM-generated code?
- Can I run arbitrary shell commands in a managed sandbox?
- How do I persist sandbox state between LLM calls?
- How do I get pandas DataFrames and plots back into the LLM context?
- What is the canonical "code interpreter" architecture?
- How do I let a coding agent run pytest or cargo test in a clean environment?
- How do I deploy a Jupyter-like sandbox per user session?

### Problem Statements

- Running untrusted LLM-generated code on your host is unsafe
- Self-managing Firecracker micro-VMs and templates is operationally heavy
- LLM tool outputs need structured results (logs, plots, dataframes), not raw strings
- Long agent sessions cost compute while idle without snapshots
- Cold-start latency matters for spiky traffic; pools and warm sandboxes add complexity
- Filesystems vanish on kill unless externally persisted
- Per-sandbox metered compute can run away on long unsnapshotted sessions

### When to Pick This

- Pick E2B when your agent needs to run code (Python/TS/Rust) and observe structured results — this is the canonical code-writing/data-analysis surface
- Pick E2B over a local Docker container when you need millisecond boot, snapshots, and Firecracker-grade isolation as a managed service
- Pick E2B over a raw VM when you want first-class agent integrations (OpenAI Agents SDK, LangChain/LangGraph, Anthropic, Vercel AI, CrewAI, Pydantic AI)
- Pick Browserbase instead when the action surface is web pages (browser automation), not arbitrary compute
- Pair with MCP by wrapping `runCode` as an MCP tool so any MCP host can use it
- Self-host E2B when data-locality or compliance rules forbid managed cloud

### Related Terms and Aliases

- Environment to Build
- code interpreter as a service
- sandboxed Python kernel
- agent VM
- Firecracker sandbox
- Jupyter kernel for agents
- managed code execution
- LLM tool execution sandbox
- agent compute surface
