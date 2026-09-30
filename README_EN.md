# PxDCA

**Language: [Traditional Chinese (default)](README.md) | English**

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)

> **Before implementation, align the logic.**
>
> Before implementation begins, use written evidence to confirm that the requirements, technical plan, and delivery basis point in the same direction.

> **Rename notice:** This project was formerly named `LogicMCP_Server` and is now officially named `PxDCA`. Current names, settings, and paths use PxDCA. Legacy `MCP_*` environment variables remain temporarily readable for compatibility and emit deprecation warnings. Other old names remain only in historical or migration documentation.

PxDCA is a written-planning and boundary-alignment service built with [FastMCP 3](https://gofastmcp.com/). It does not write the final product code. Through MCP and its internal Prompt TXT files, it guides AI to establish requirements and technical plans that are evidence-based, traceable, and resistant to unauthorized scope expansion. The first three public MCP Tools produce a requirements specification, an architecture specification, and a PM/PG alignment report with a responsibility handoff recommendation. The fourth Tool produces an optional `SKILL.md`. See [PDCA for Responsible AI](https://mydrego-james.github.io/) for the project philosophy and related tools.

- `generate_requirements`: starts or resumes a requirements interview and produces a requirements specification when complete.
- `generate_architecture`: produces a technical-selection and architecture specification from a completed requirements session.
- `run_audit`: uses the PQ perspective to check consistency, coverage, risk, and traceability between PM requirements and the PG technical plan.
- `generate_skill`: exports the canonical `SKILL.md`, optionally adding contextual guidance without replacing its core rules.

For the full project background and the earlier LogicMCP story, see the [PxDCA Wiki](https://github.com/mydrego-James/PxDCA/wiki).

## 1. Story

AI can rapidly generate code, documents, architecture, and tests. The software-development bottleneck has therefore shifted from output speed to direction alignment and boundary control. The faster execution becomes, the more expensive rework becomes when the direction is wrong, responsibilities cross their boundaries, or task levels are mixed together.

The common problem is not that work cannot be produced. It is that:

- implementation begins before requirements are confirmed;
- the requester, developer, and AI understand the target differently;
- AI fills gaps with plausible but unconfirmed conditions;
- a local feature is correct while the overall business logic drifts from its original purpose;
- a large plan is not decomposed into verifiable tasks, forcing one executor to handle global architecture and low-level details at the same time;
- after work is delegated, the executor sees only a fragment and loses the parent purpose, context, and constraints; or
- the result has no falsifiable quality gate, so it appears complete without being verifiable.

PxDCA does not turn PDCA into a mandatory pipeline, and it does not replace the requester, project manager, architect, developer, or AI. Here, PDCA is a judgment discipline AI must retain throughout its work: establish purpose, develop a response from evidence, challenge the connection between purpose and response, and then accept, revise, hold, or hand off responsibility. This discipline is already present from Q0 and the QA stages onward.

Two dimensions must remain separate:

- **MCP FLOW** has ordered dependencies: a requirements baseline must exist before a technical response can be produced, and both must exist before alignment can be checked.
- **PDCA discipline** operates within every working state, but it does not require each state to produce separate P, D, C, and A documents or force AI to mechanically narrate four steps.

The `x` in PxDCA represents a polymorphic context. It may express a role focus, task level, domain focus, or responsibility-handoff state. Its two clearest current forms are:

1. **Working focus: PM, PG, and PQ**

   - `PM`: collects requirements, clarifies the target and scope, and establishes the written requirements baseline.
   - `PG`: receives confirmed PM evidence and creates the written technical-selection and architecture response without expanding business requirements.
   - `PQ`: aligns the upstream and downstream evidence between PM and PG. It checks whether technical decisions have requirement sources, whether requirements receive technical responses, and where a gap should be corrected. PQ is not another independent planning project.

2. **Task handoff expansion: `P1 / D1 / C1 / A1 → P2 ...`**

   The purpose, evidence, decisions, and checks produced by one task may become input to the next task. This requires traceable responsibility handoff. It does not mean that the MCP Server must execute a rigid PDCA state machine, nor that every `P_x` must complete its own DCA.

Purpose and response form a **Purpose × Response** relationship: one purpose may require several responses, and one response may support several purposes. PM, PG, and PQ are switchable working states rather than permanent job titles. The same AI may switch between them only while preserving the active boundary and evidence source.

The current core relationship is:

```text
PM: requirements collection and written requirements planning
          │ evidence-backed requirements baseline
          ▼
PG: technical selection and written architecture planning
          │ evidence linking requirements and technical decisions
          ▼
PQ: check the PM ↔ PG connection and any deviation
          │ disposition, next purpose, evidence, risks, and recommended owner
          ▼
Handoff: responsibility-transfer recommendation; not a fifth Tool and does not
         automatically create another session
```

Only PM and PG produce the primary planning content. PQ aligns them and identifies gaps, correction routes, and Handoff. The process produces written artifacts and uses sources, answers, decisions, and mappings as evidence so AI does not invent unauthorized requirements or technical scope.

The current release focuses on written planning before software, website, API, data, integration, and feature development. Reports, RAG, and knowledge-management systems are possible extensions of the polymorphic model, but they are not capabilities promised by the current built-in templates.

> AI moves work forward quickly; PxDCA keeps its boundaries and engineering direction aligned.

## 2. Installation

### 2.1 Local deployment

Requirements:

- Git
- Windows, macOS, or Linux
- Python 3.13 recommended, or Python 3.12

Clone the project:

```powershell
git clone https://github.com/mydrego-James/PxDCA.git
cd PxDCA
```

Non-secret runtime settings are centralized in `config/pxdca.toml`. To create a local configuration:

```powershell
Copy-Item config/pxdca.example.toml config/pxdca.local.toml
$env:PXDCA_CONFIG = "$PWD/config/pxdca.local.toml"
```

Install and start on Windows:

```powershell
.\install.bat
.\run.bat
```

Install and start on macOS or Linux:

```bash
chmod +x install.sh run.sh
./install.sh
./run.sh
```

Default MCP endpoint:

```text
http://127.0.0.1:8000/mcp
```

The installer accepts Python 3.12 or 3.13, creates `.venv`, installs the pinned FastMCP 3 dependencies, and runs basic checks. `state.root` stores requirements-interview state. `artifacts` controls optional file outputs. They serve different purposes.

### 2.2 Docker installation

Requirements: Docker Engine, plus Docker Compose when using Compose.

```powershell
git clone https://github.com/mydrego-James/PxDCA.git
cd PxDCA
docker compose up --build -d mcp
```

Inspect status and logs:

```powershell
docker compose ps
docker compose logs -f mcp
```

Stop the service:

```powershell
docker compose down
```

Compose uses separate named volumes for state, artifacts, and logs. The state volume preserves interview state, so an existing `session_id` remains resumable after the container is rebuilt. `docker compose down -v` deletes these volumes; do not use `-v` unless the stored data is no longer needed.

To change the exposed IP address, port, or configuration file, edit `.env`:

```dotenv
PXDCA_BIND_HOST=127.0.0.1
PXDCA_HOST_PORT=8000
PXDCA_CONFIG_FILE=./config/pxdca.toml
```

See [docker.md](docker.md) for the complete configuration and `docker run` examples.

### 2.3 Glama directory and releases

The maintainer has claimed the [PxDCA MCP Server page on Glama](https://glama.ai/mcp/servers/mydrego-James/PxDCA). The root [`glama.json`](glama.json) provides Glama maintainer metadata and contains no API key, token, or deployment credential.

A Glama Release is a container version built, tested, and published by Glama; it is different from a GitHub Release. The Glama build specification should use Python 3.13, install `requirements.txt` in an isolated virtual environment, and start PxDCA over stdio:

```json
{
  "build_steps": [
    "uv venv /app/.venv --python 3.13",
    "uv pip install --python /app/.venv/bin/python -r requirements.txt"
  ],
  "cmd_arguments": ["/app/.venv/bin/python", "-m", "server.fastmcp_service"],
  "environment": {
    "PXDCA_TRANSPORT": "stdio"
  }
}
```

The repository `Dockerfile` remains the self-hosted Streamable HTTP deployment. Glama wraps the stdio command above with `mcp-proxy` to provide its hosted connection. Both deployment modes use the same Server and four public Tools, but their transport boundaries differ.

### Credentials and API boundary

The repository, Docker image, and example configuration do not contain the maintainer's test API, tokens, or credentials. Controlled text generation is provided through MCP Client Sampling and never uses the maintainer's model API account. Anyone adding a domain, authentication layer, external API, or another service must configure their own uncommitted `.env`, platform Secret, or Secret Manager values in their deployment environment.

## 3. Usage: VS Code example

### 3.1 Connect PxDCA

Start the PxDCA Server, then create `.vscode/mcp.json` in the VS Code workspace that will use it:

```json
{
  "servers": {
    "pxdca": {
      "type": "http",
      "url": "http://127.0.0.1:8000/mcp"
    }
  }
}
```

Then in VS Code:

1. Open the Command Palette (`Ctrl+Shift+P`).
2. Run `MCP: List Servers`.
3. Start `pxdca`.
4. Confirm that all four PxDCA Tools appear in the Chat tool list.

The MCP Client used by VS Code must support MCP Sampling because the Server asks the Client model to perform controlled reasoning inside the bounded workflow.

### 3.2 Create a requirements specification

Describe the target directly in Chat:

```text
Use PxDCA to create a requirements specification for an equipment maintenance
management system. Use the professional profile and confirm each question with me.
```

The AI calls:

```json
{
  "tool": "generate_requirements",
  "arguments": {
    "q0": "Build an equipment maintenance management system",
    "profile": "professional"
  }
}
```

The Server returns the `session_id`, current question, and interview state. To answer, the AI calls the same Tool:

```json
{
  "tool": "generate_requirements",
  "arguments": {
    "session_id": "SERVER_RETURNED_SESSION_ID",
    "answer": "Maintenance supervisors and field technicians will use it."
  }
}
```

If the conversation is interrupted, retain the `session_id` and later request:

```text
Resume the previous requirements interview with session_id
"SERVER_RETURNED_SESSION_ID".
```

When only `session_id` is supplied, the Server loads the last validated state and returns the current question without repeating accepted answers.

### 3.3 Create an architecture specification

After the requirements specification is complete, request:

```text
Use the same PxDCA session to create the architecture specification.
```

Corresponding call:

```json
{
  "tool": "generate_architecture",
  "arguments": {
    "session_id": "SERVER_RETURNED_SESSION_ID"
  }
}
```

The requirements and technical-architecture stages share the same `prompts/profiles/*.txt` capability definitions. The requirements stage may switch focus by question. The architecture stage reads the complete first-stage questions, answers, assessments, and `capability_profile` values, performs one overall focus alignment, and merges the relevant TXT definitions into one AI capability boundary in the architecture Prompt. It does not switch identity question by question again. The stages still use different Prompt contracts and Markdown rendering templates, so they produce separate requirements and architecture documents rather than mixing both into one template.

### 3.4 Run the audit

After the architecture specification is complete, request:

```text
Audit the requirements and architecture for this session. List omissions,
conflicts, risks, and traceability results.
```

Corresponding call:

```json
{
  "tool": "run_audit",
  "arguments": {
    "session_id": "SERVER_RETURNED_SESSION_ID"
  }
}
```

Completed architecture and audit calls are idempotent. Reusing the same `session_id` returns the existing artifacts instead of generating them again.

## 4. Current architecture

PxDCA exposes three written-artifact operations and one Skill-generation Tool to the MCP Client. MCP Tools are the invocation interface. The internal Prompt TXT files teach AI the PM, PG, and PQ boundaries and the PDCA judgment discipline. A shared PDCA judgment prompt is injected into the requirements, technical-planning, and PQ audit stages and explicitly prevents it from becoming a second workflow or mandatory four-step narration. Policies, Profiles, Schemas, Validators, state transitions, and Renderers are also private implementation details; they are not registered as additional MCP Prompts, Resources, or Tools.

```text
VS Code / another MCP Client
        │
        │ MCP + Client LLM Sampling
        ▼
┌─────────────────────────────────────────────┐
│ PxDCA Server                                │
│                                             │
│  generate_requirements                      │
│  generate_architecture                      │
│  run_audit                                  │
│  generate_skill                             │
│                │                            │
│                ▼                            │
│  Private workflow orchestration             │
│  Prompts / Policies / Schemas / Validators  │
│  State transitions / Renderers              │
└───────────────────┬─────────────────────────┘
                    │
                    ▼
          Persistent session + artifacts
```

### Workflow

```text
Q0
 ↓
generate_requirements
 ↓ question-by-question interview, validation, and persistent state
PM requirements specification
 ↓
generate_architecture
 ↓ technical selection and planning bounded by PM evidence
PG architecture specification
 ↓
run_audit
 ↓ align PM and PG from the PQ perspective
Alignment and audit report
 ↓
Handoff: ready / revise PM / revise PG / user decision required / hold
```

This order is the document-dependent MCP FLOW. PDCA is the judgment discipline used within every stage to establish purpose, develop a response, check evidence, and decide the next action. Handoff is structured output from `run_audit`; it is not another public Tool and does not automatically create another session.

An MCP connection is not workflow state. The Server stores only validated state and uses `session_id` to resume work. Sessions default to `data/state/`; optional requirements, architecture, and audit files are stored under `data/artifacts/<session_id>/`; Server logs are written to `data/logs/`. All locations can be overridden through `config/pxdca.toml` or `PXDCA_*` environment variables.

### Project structure

```text
server/
├─ entrypoint.py
└─ fastmcp_service/
   ├─ public_tools.py       four public MCP Tools
   ├─ skill_service.py      SKILL.md generation and optional optimization
   ├─ workflow_service.py   workflow and persistent sessions
   ├─ prompts.py            private Prompt loading and composition
   ├─ prompts/              private Prompt templates
   ├─ resources.py          private Resource loading
   ├─ resources/            Policies, Schemas, and Templates
   ├─ tools/                private Validators and Renderers
   ├─ tests/                public-workflow tests
   └─ docs/                 contracts, boundaries, and workflow documents

logs/                       Server runtime logs
config/                     external PxDCA runtime configuration
data/                       runtime state, artifacts, and logs; not committed
tools/                      maintenance tools and development plans
└─ SKILL.md                 canonical AI Skill template

install.bat                 local installation
run.bat                     local startup
fastmcp.json                FastMCP filesystem deployment
Dockerfile                  Docker image
compose.yaml                Docker Compose service
```

See [MAP.MD](MAP.MD) for the complete file map and [IO_BOUNDARY.md](server/fastmcp_service/docs/IO_BOUNDARY.md) for input/output boundaries.

## 5. Optional tool: SKILL.md

[tools/SKILL.md](tools/SKILL.md) is the canonical template that can be provided independently to an AI. It has only three responsibilities:

1. Teach AI the relationship and distinction between the original `Plan → Do → Check → Act` and PxDCA's AI-planning and responsibility-handoff interpretation: `Problem/Purpose → Design/Develop response → Check/Challenge → Action/Assume responsibility`. They share a judgment discipline but do not use identical workflow meanings.
2. Inspect whether the available requirements, specifications, plans, and architecture are sufficient for the requested work instead of trusting filenames alone.
3. When information is insufficient and the user agrees, use `generate_requirements`, `generate_architecture`, `run_audit`, and the `session_id` continuation rules correctly.

The Skill is not a persistent service, background monitor, workflow engine, or required MCP dependency. Users may call MCP directly or provide `SKILL.md` to an AI. The first three planning workflows do not depend on the Skill.

### The Skill is not mandatory

Installing and starting PxDCA and using its first three planning workflows do not require the Skill. Depending on the IDE, Agent, web Chat, or development practice, a user may:

- call PxDCA Tools directly;
- use custom Prompts or Agent rules;
- reference `SKILL.md` in a platform that supports Skills; or
- not use a Skill at all.

`SKILL.md` prepares AI to understand the relationship and distinction between the two PDCA interpretations, judge whether the current state is sufficient, and use PxDCA MCP Tools correctly when information is missing. It also separates MCP FLOW from PDCA discipline and does not require PM, PG, and PQ to produce their own DCA document sets.

### Load the Skill in an AI tool

IDEs, Coding Agents, and web Chats support Skills in different ways. There is no universal installation or invocation standard, so follow the instructions for the actual platform.

In an environment that supports invoking a Skill by name, use the Skill's own name. The canonical template is named `pxdca-pdca`, so it may be referenced before the main task like this:

```text
/pxdca-pdca

Inspect the current project and determine whether its requirements,
specifications, and architecture are sufficient to begin development.
```

`/pxdca-pdca` is an example of invoking this template by Skill name, not a command standardized across every platform. If the frontmatter `name` is changed, invoke `/<custom-skill-name>`, such as `/my-project-pdca`. A platform may instead load a Skill through a menu, mention, attachment, or another interface.

### Relationship between MCP and Skill

From the perspective of giving AI capabilities, MCP can also be considered a broad kind of Skill. The primary difference is where the capability is placed and loaded:

```text
MCP
→ capabilities, Tools, and state live on a local or remote MCP Server
→ AI connects and invokes them through the MCP protocol

SKILL.md
→ instructions and usage knowledge are loaded into a local environment or sandbox
→ when name invocation is supported, use /pxdca-pdca or /<custom-skill-name>
```

In PxDCA, the MCP Server provides executable requirements, architecture, audit, Handoff, and Skill-generation capabilities. `SKILL.md` teaches AI in its local or sandbox context how to understand the two PDCA interpretations, inspect state, and invoke MCP correctly when needed. They can be used together or separately according to the environment.

If an IDE or web Chat does not support Skills, upload, attach, or paste `SKILL.md` and explicitly ask the LLM to read it first:

```text
Read the attached SKILL.md first. Confirm its two PDCA interpretations and
PxDCA usage rules, then inspect whether this project has a sufficient
development baseline.
```

Adding Markdown to an existing conversation is not a standardized Skill-loading mechanism. Existing messages, other prompts, and context order may influence the LLM's interpretation, so this is generally less reliable than a platform-native Skill reference. For important work, confirm that the AI understands the Skill's three responsibilities before continuing.

The fourth public Tool, `generate_skill`, can export a Skill:

```json
{
  "tool": "generate_skill",
  "arguments": {
    "mode": "template"
  }
}
```

`template` returns the canonical template unchanged. `optimized` uses Client LLM Sampling to append contextual guidance:

```json
{
  "tool": "generate_skill",
  "arguments": {
    "mode": "optimized",
    "customization": "Add the current team's document-review rules."
  }
}
```

Contextual content may only be appended. It cannot replace the two PDCA interpretations, Purpose × Response, Handoff, the first three Tool contracts, or the `session_id` rules. `generate_skill` does not create a requirements session.

`generate_skill` always returns the Skill content. It additionally writes a `SKILL.md` file only when `artifacts.enabled=true`. It does not install, register, or enable a Skill in an IDE, Agent, or Chat. The generated result must still be loaded through the actual platform:

```text
generate_skill
→ receive content and, when artifacts are enabled, artifact.path
→ install it in the platform's designated Skill location
→ invoke /pxdca-pdca or /<custom-skill-name> when name invocation is supported
→ explicitly provide the Markdown to the LLM when Skills are unsupported
→ only then can the AI use the Skill to inspect state and decide whether to use MCP
```

A local MCP Client can usually access `artifact.path` directly. When PxDCA is remotely deployed, that path belongs to the Server filesystem. The Client should save or load the `content` returned by the Tool and must not assume it can open the remote Server path.

## 6. Roadmap

The current PxDCA Server is an executable written-planning front line. Through PM requirements collection, PG technical planning, and PQ alignment, it tests whether AI can continue working from evidence without unauthorized expansion. Planned directions include:

1. **Validate and improve the optional `SKILL.md`**

   Use real AI cases to test whether AI understands PDCA as a continuing judgment discipline rather than a mechanical workflow.

2. **Strengthen PM requirement evidence and boundaries**

   Make each requirement traceable to Q0, accepted QA answers, confirmed assumptions, and user authority so an AI recommendation cannot silently become a requirement.

3. **Strengthen PG technical-selection mapping**

   Map technical choices, module responsibilities, and architecture decisions to PM requirements. Technical planning may expand only within authorized requirement scope.

4. **Validate PQ Handoff in real cases**

   PQ now outputs a disposition, next purpose, required actions, evidence, unresolved risks, and recommended owner. Real cases will validate whether `ready_for_handoff`, revise PM, revise PG, user decision required, and hold are clear enough.

5. **Research parent-child sessions across tasks**

   Handoff currently records only a responsibility-transfer recommendation and does not automatically create another session. Future work may preserve parent-child relationships across `P1 / D1 / C1 / A1 → P2 ...` without hardening PDCA discipline into a state machine.

6. **Expand Profiles, Templates, Clients, and deployment validation**

   First validate capability TXT files and requirements and architecture templates across software domains, plus MCP Clients, models, and Agent runtimes beyond VS Code. Cross-domain templates for reports, RAG, and knowledge management remain later extensions and are not claimed as completed capabilities.

7. **Evaluate FastMCP 4 on a separate branch**

   `main` remains fixed on FastMCP 3.x. FastMCP 4 sampling and interface migration will be evaluated only on a separate branch rather than maintaining two compatibility paths in the current mainline.

PxDCA does not aim to become a code generator or mandatory workflow engine. It starts from traceable written planning so AI retains PDCA judgment and boundary awareness during requirements collection, technical selection, and upstream/downstream alignment.

---

GitHub preserves code and version history. PxDCA preserves the requirements, plans, boundaries, and checks behind the result.

## License and public scope

The code, Prompt TXT files, templates, and documentation actually published in this repository are licensed under the [Apache License 2.0](LICENSE). Subject to that license, anyone may use, modify, fork, distribute, and commercially use the published material. Feedback, corrections, and collaborative development are welcome. Unless separately agreed in writing, contributions intentionally submitted to this project are provided under the same license.

This open-source license covers only content actually published in this repository. Unpublished customer data, organization-specific Prompts, custom templates, internal processes, trade secrets, patented technology, and individual contract deliverables are outside this repository and its open-source license. They are governed separately by the applicable contract, confidentiality agreement, and individual license terms.
