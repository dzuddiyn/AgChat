# AgChat

**AgChat** is the project space and Source of Truth (SoT) for research, architecture decisions, validation plans, and implementation notes concerning agentic coding through an AI chat interface.

The operating principles are **reuse before rebuild** and **PoC first, practical security**: use existing capabilities, prove a usable workflow quickly, and apply safeguards proportionate to the actual development environment. Do not add security layers that do not materially reduce a relevant risk.

## Current direction

The validated V1 baseline is **ChatGPT Chat Mode + MCP SuperAssistant + Desktop Commander**:

`ChatGPT Go (same conversation) → MCP SuperAssistant browser extension → local MCP proxy → Desktop Commander → disposable local workspace`

The objective is to prove that the original ChatGPT conversation can repeatedly discover and invoke local tools, read and modify files, run a controlled test, and receive verifiable results (stdout, stderr, exit code, Git status/diff) without duplicate execution or loss of conversation continuity.

**Do not install or integrate ChatCMD or CoS now.** TEST A has passed for the agreed V1 PoC scope; evaluate another runtime only if a concrete implementation task exposes a gap.

## Start here

- [Agent instructions (AGENTS.md)](AGENTS.md)

- [Detailed research, architecture decisions and roadmap](docs/AGCHAT-RESEARCH-AND-PLAN.md)
- [SuperAssistant development method on Xubuntu: RDC, CDP, GUI and MCP setup](docs/AGCHAT-WORKING-METHOD.md)
- [TEST A execution and acceptance plan](docs/TEST-A-PLAN.md)
- [TEST 43 — SuperAssistant proxy + Orchestrator E2E result](docs/TEST-43-PROXY-ORCHESTRATOR-RESULTS.md)
- [TEST 44 — controlled command, failure and duplicate guard](docs/TEST-44-COMMAND-FAILURE-DUPLICATE-RESULTS.md)
- [TEST 45 — workspace filesystem boundary](docs/TEST-45-WORKSPACE-BOUNDARY-RESULTS.md)
- [TEST 46 — practical coding loop](docs/TEST-46-PRACTICAL-CODING-LOOP-RESULTS.md)
- [TEST 47 — ChatGPT long-paste threshold](docs/TEST-47-LONG-PASTE-THRESHOLD.md)

## Project boundaries

- ChatGPT Chat Mode is the AI reasoning/control surface; local tools provide the hands and feet.
- The initial target is one task in one full ChatGPT conversation/session.
- Open Browser Agent (OBA) is outside AgChat scope.
- Do not build a new reviewer, moderator, coordinator, worker pool, task manager, trace system, approval ledger, or orchestration framework when an existing product already provides the required capability.
- Multi-agent, independent reviewer, personal AI workforce, and automatic resume are not MVP requirements.
- Do not use multiple chats, agents, or tools to evade provider usage limits, entitlements, or safety policies.
- The Xubuntu PC is dedicated to development and experimentation. Host-level command execution is acceptable for the PoC when actions are understood and non-destructive; do not require a container/VM solely as a formality.
- Keep sensible safeguards: avoid destructive system/disk commands, protect credentials and important data, and inspect Git changes. Add stronger isolation only when a concrete risk or use case justifies it.

## Status

- Research and architecture direction: documented from the available project discussions and prior decisions.
- TEST A: **PASS for the agreed V1 PoC scope, with documented limitations.** Session 0/1/2 PASS; Test 43 proxy E2E PASS; Test 44 controlled execution PASS (duplicate exit-status telemetry incomplete); Test 45 filesystem allow-list boundary PASS; Test 46 practical coding loop PASS (four unit tests, exit 0, read-back verified). Generic transport exactly-once, proxy reconnect recovery and OS-level shell isolation are not claimed. Test 47 also identified a ChatGPT composer clipboard-paste threshold: 9,999 characters remains inline, while 10,000 becomes a text attachment; use an 8,000-character conservative paste ceiling.

- ChatCMD / CoS: **deferred unless the current validated workflow exposes a concrete gap**.
- Permanent Xubuntu installation or SSD formatting: **not authorised by this documentation**; wait for an explicit user decision.

## Source of Truth policy

This repository records accepted decisions, constraints, research findings, test evidence, and changes in status. Distinguish clearly between:
- **Confirmed** — directly verified by a test or authoritative source.
- **Accepted decision** — chosen direction, not necessarily proven in operation.
- **Hypothesis / pending validation** — plausible but not yet demonstrated.
- **Deferred / out of scope** — deliberately not part of the current phase.

Do not convert a planned test into a claimed success. Update the detailed documents when evidence changes the decision.
