# AgChat — Research, Architecture Decisions & Roadmap

**Document status:** Consolidated project record  
**Baseline date:** 27 November 2026  
**Repository:** https://github.com/dzuddiyn/AgChat  
**Purpose:** Preserve the research, reasoning, accepted decisions, constraints, and implementation plan discussed for AgChat.  
**Important:** This document distinguishes decisions from verified test results. TEST A is in progress; connection and basic filesystem operations have passed, while Git verification and end-to-end command workflow remain pending.

---

## 1. Executive summary

AgChat explores a practical form of **agentic coding through an AI chat interface**. The AI remains in its normal chat conversation and acts as the reasoning/control surface. Local tools provide access to the user's development environment: files, terminal commands, tests, Git, and eventually optional worker/runtime capabilities.

The central idea is not to build another autonomous-agent platform. It is to connect an existing AI chat experience to existing local execution tools while preserving:

1. The identity and continuity of the original conversation.
2. Explicit control over local filesystem and command execution.
3. A verifiable path from AI tool request to local execution and returned result.
4. A clear security boundary around the workspace and host machine.
5. A low-resource, maintainable setup that can run on the available Xubuntu machine.

The agreed principle is **reuse before rebuild**. Existing projects such as Agent360, MCP SuperAssistant, Desktop Commander, ChatCMD, and CoS should be evaluated for the exact capabilities they already provide. AgChat should not duplicate their orchestration, trace, approval, or worker-management features.

The immediate gate is **TEST A: MCP SuperAssistant + Desktop Commander**. ChatCMD and CoS are not to be installed or integrated until TEST A demonstrates that ChatGPT Go can reliably read, change, and verify local files through the same conversation.

## 2. Terminology and project identity

The discussions used several related names. Preserve these distinctions to avoid mixing separate implementations:

- **AgChat** — the general concept and this repository's project/SoT space: agentic coding through chat mode.
- **AgChat360** — the Agent360-based implementation/development track discussed as the project-specific evolution of Agent360-Hardened.
- **Agent360 / Agent360-Hardened** — an existing implementation/reference to audit and reuse, not a reason to rebuild the same runtime from scratch.
- **AgChatSteroid** — an earlier, separately named direction involving ChatGPT Go with CoS. It was explicitly separated from the Agent360-based work in the earlier naming discussion. CoS remains a candidate for later evaluation, not the current first implementation.
- **OBA (Open Browser Agent)** — explicitly outside AgChat scope.

The repository name remains **AgChat** and is the source of truth for the consolidated research and decisions. Where an implementation-specific name is needed, use AgChat360 for the Agent360-based track rather than silently conflating every experiment.

## 3. Problem statement and intended outcome

### 3.1 Problem

An AI chat model can reason about code, plan changes, and generate patches, but its usefulness for real agentic coding depends on a reliable execution bridge into a local development environment.

The project needs to establish whether an ordinary ChatGPT Go conversation can:
- Discover available local tools.
- Read files from a deliberately scoped workspace.
- Create and edit files.
- Run a controlled command or test.
- Return complete execution evidence to the same AI conversation.
- Continue through multiple sequential tool calls without losing context or executing the same action twice.
- Inspect Git status and diffs to verify what changed.

### 3.2 Intended outcome

A working, controlled loop:

`User ↔ ChatGPT conversation ↔ browser-side MCP bridge ↔ local MCP proxy ↔ local execution tool ↔ disposable workspace`

The AI decides what to do and interprets the result. The local execution layer performs only the permitted operations and returns evidence. The user remains in control of approvals and the security boundary.

### 3.3 MVP scope

The initial scope is deliberately narrow:

- One task at a time.
- One full ChatGPT conversation/session as the baseline.
- Sequential tool calls in that same conversation.
- A local workspace with disposable test files.
- File read/write and Git verification.
- Controlled command/test execution only after the security boundary is established.

The baseline is not a general-purpose multi-agent workforce. Additional worker conversations may be used in later experiments, but they do not replace the same-chat baseline.

## 4. Accepted architectural principles

These are accepted project constraints and should not be reopened without new evidence or a material change in requirements.

### 4.1 Chat-first, not runtime-first

- ChatGPT Chat Mode is the AI reasoning and interaction surface.
- The chat conversation is the primary user-facing session.
- Local tools act as the AI's execution capabilities.
- The initial proof must preserve the original conversation rather than silently moving the task to a separate agent interface.

### 4.2 Reuse before rebuild

Do not create custom versions of capabilities already supplied by existing products. In particular, do not build:
- A new reviewer/moderator.
- A new orchestration framework.
- A task manager or coordinator.
- A worker pool.
- A custom trace or tool-call history system.
- A duplicate approval ledger.
- A duplicate task timeline or execution log.
- A custom resume system.

Use the native feature of the chosen product where it is available and suitable. Build only a small adapter or integration layer when a proven interoperability gap requires it.

### 4.3 No policy or entitlement circumvention

The bridge is an integration experiment, not a way to bypass provider usage limits, account entitlements, safety policies, or product restrictions. Do not distribute one task across multiple chats or agents to evade those controls.

### 4.4 PoC first, practical security (accepted standing principle)

The development PC is dedicated to development and experimentation. The project must not become blocked or unnecessarily complicated by production-grade security requirements that do not materially improve this PoC.

- Prioritise a working, usable end-to-end PoC.
- Use the simplest existing tool path that proves the workflow; do not require a container or VM merely to satisfy a formal checklist.
- Host-level command execution on the dedicated development PC is acceptable for the PoC when the action is understood and reasonably safe.
- Keep proportionate safeguards: avoid destructive disk/system commands, protect credentials and important personal/production data, and inspect Git status/diffs.
- Keep synthetic/disposable test data where convenient, but do not add elaborate isolation or approval layers without a concrete risk.
- Introduce stronger isolation later if real usage, a specific workload, or a demonstrated risk warrants it.

Security remains important, but it is an enabler of practical development rather than a reason to prevent the PoC from working.

## 5. Existing Agent360 capability audit

Agent360-Hardened is an existing implementation/reference and should be inspected before writing replacement code.

The reviewed design already contains or references:
- A same-session tool-call loop.
- A state machine with states such as `IDLE → CALL_DETECTED → MCP_EXECUTING → INJECTING → WAITING_AI`.
- Tool-call parsing/detection.
- MCP integration.
- Queue/correlation mechanisms.
- Result injection back into the AI conversation.
- Trace/execution history.
- Workspace concepts.
- Filesystem operations.
- Shell, test, lint and Git-related capabilities in the broader implementation.
- Templates and supporting workflow elements.
- Hardened security concepts such as deny-by-default behaviour, approvals, and an approval ledger.

The Hardened profile/configuration discussed in the audit disables or restricts higher-risk capabilities such as shell execution, tests/lint, package installation, and process launching. Exact enabled capabilities depend on the checked-out version and configuration; verify source and runtime behaviour rather than assuming that a feature listed in the broader project is enabled in the hardened profile.

**Implication:** AgChat should reuse or adapt the existing implementation where appropriate. Do not recreate its state machine, parser, queue, correlation, injection, trace, workspace, or Git workflow merely to give them new names.

## 6. Host-only vs Host + Container vs Host + VM

The deployment options were assessed against actual resource constraints and the fact that the machine is intended for work, not as a disposable high-performance lab.

### 6.1 Host-only

**Description:** The bridge and local MCP tools run directly on Xubuntu under the normal host OS user.

**Advantages**
- Lowest setup and runtime overhead.
- Simplest path to a first proof.
- Direct access to browser, filesystem, and local applications.
- Fewer moving parts and easier debugging.

**Risks**
- A local tool running as the user may inherit broad OS permissions.
- Filesystem allow-lists do not automatically constrain terminal commands.
- A command can affect files or processes outside the intended workspace unless the process itself is constrained.
- Browser extension, local proxy, and local tool permissions become part of the host's attack surface.

**Assessment:** Suitable for the initial read-only and file-operation proof. Do not enable unrestricted shell execution merely because the setup is host-only and the user can approve calls.

### 6.2 Host + Container

**Description:** Xubuntu remains the host; command/test execution is delegated to a container with only the disposable workspace mounted.

**Advantages**
- Better containment for build/test commands.
- Can limit mounted paths and avoid exposing the full home directory.
- More reproducible test environment.
- Lower overhead than a full VM in many configurations.

**Risks**
- Docker/Podman installation and permissions add complexity.
- Containers are not automatically secure; privileged mode, broad mounts, host sockets, or excessive capabilities can defeat isolation.
- Browser and desktop interaction generally remain on the host.
- Resource use and image storage still matter on a constrained machine.

**Assessment:** Preferred next security boundary if TEST A reaches command execution and the host has sufficient resources. Use a non-privileged container, mount only the test workspace, avoid host sockets and broad home-directory mounts, and keep network access disabled or restricted unless the test requires it.

### 6.3 Host + VM

**Description:** Xubuntu host runs a separate guest OS for the execution environment.

**Advantages**
- Stronger separation of guest and host than a normal host process.
- Useful for risky or incompatible workloads and snapshot/rollback experiments.
- Can test a separate OS environment.

**Risks**
- Highest CPU, memory, and storage overhead.
- More configuration and maintenance.
- May compete with other intended workloads, including Home Assistant OS VM, Docker, Temaya, and database services.
- The PC is work-dedicated and has limited performance headroom; a VM-first design risks making the everyday workflow unreliable.

**Assessment:** Not required for V1. Keep as a later option only if container isolation proves insufficient or a specific test requires a separate guest OS.

### 6.4 Hardware facts and limits

A previous hardware assessment recorded an Intel Core i7-4710HQ (4 cores / 8 threads), approximately 15 GiB RAM, a 223.6 GB SSD and a 465.8 GB HDD, with no swap at the time of inspection. **Those figures must not be attributed to Niniy-PC.** The user clarified that Niniy-PC is the Mini PC and is not the Core i7 laptop. The precise Niniy-PC specifications and available persistent storage must be inventoried on the actual machine before choosing container/VM settings.

Xubuntu was being tested from a normal Live USB without persistence. Therefore, installed packages, configuration, and free space on the intended SSD were not yet established by that live session.

### 6.5 V1 deployment decision

Start with the simplest host-side integration and prove the workflow directly on the development PC. Host-only command execution is acceptable for this PoC; use ordinary care around destructive commands and important data. A container may be added later if it provides a concrete benefit, but it is not a prerequisite. VM deployment is not required for V1.

## 7. Current candidate integration: TEST A

### 7.1 Target flow

`ChatGPT Go`
→ `MCP SuperAssistant browser extension`
→ `local MCP proxy`
→ `Desktop Commander MCP`
→ `disposable local workspace on Xubuntu`

Tool results must be returned to the same ChatGPT conversation that initiated the call.

### 7.2 Why this is first

This is the smallest experiment that can answer the project's most important unresolved question: can the selected ChatGPT Go/browser setup reliably invoke local MCP tools and receive their results without requiring a custom agent runtime?

It isolates the core bridge before adding child agents, worker routing, or a second execution framework.

### 7.3 MCP SuperAssistant

The reviewed project material describes a browser extension plus a local proxy that can connect browser-side chat interactions to local MCP servers. The integration path discussed supports local MCP server transports including SSE and Streamable HTTP.

Example endpoints encountered in the reviewed setup:
- SSE: `http://localhost:3006/sse`
- Streamable HTTP: `http://localhost:3006/mcp`

Use one transport at a time during the first test. Do not assume both endpoints are active or interchangeable in a particular version. Confirm the installed proxy's actual configuration and logs.

The extension may need to be loaded manually through the Chromium-based browser's extensions page with Developer mode and Load unpacked. Exact UI and supported browsers depend on the extension version.

### 7.4 Desktop Commander

Desktop Commander is the local MCP execution/filesystem component in the proposed test. Its filesystem allow-list (for example, an `allowedDirectories` setting) can scope filesystem tools, but it must not be treated as a complete command-execution sandbox.

Important distinction:
- Filesystem allow-list: constrains the filesystem operations that respect it.
- Terminal/process execution: may run with the operating-system user's permissions and is not automatically confined by the filesystem allow-list.
- Manual approval: allows the user to accept or reject an action but does not technically isolate the process.

Use filesystem scoping where it is already available, but do not mistake it for command isolation. For this dedicated development-PC PoC, proceed with host command execution for ordinary, non-destructive development tasks. Avoid destructive system/disk operations and protect important data. A container is optional, not a gate.

### 7.5 TEST A acceptance criteria

TEST A is a pass only when the following are demonstrated and recorded:

1. **Connection:** The extension detects the local MCP server and completes tool discovery.
2. **Read:** ChatGPT reads a known file from the disposable workspace and accurately reports its contents.
3. **Create:** ChatGPT creates a new test file in the workspace.
4. **Edit:** ChatGPT changes a known line/value in that file.
5. **Git evidence:** Git status and diff show the expected changes and no unrelated changes.
6. **Controlled execution:** A deliberately harmless command/test runs inside the chosen constrained boundary; stdout, stderr and exit code are captured and returned.
7. **Same-chat continuity:** Several sequential calls work in the original conversation without requiring a new conversation or manually copying results.
8. **No duplicate execution:** One tool request produces one execution; retries/timeouts do not silently repeat a write or command.
9. **Failure behaviour:** A missing file, invalid command, timeout, or disconnected proxy returns an intelligible failure rather than a false success.
10. **Evidence:** Commands, versions, relevant logs, Git diff, and observed results are recorded without secrets.

If any of these conditions fails, record the exact failure and adjust the bridge before moving to another runtime.

## 8. TEST A work plan

The planned work is divided into six practical sessions. The count is an estimate, not a deadline.

### Session 0 — Xubuntu readiness

Establish the actual test environment before installing anything:
- Confirm whether Xubuntu is Live USB, persistent Live USB, or installed to disk.
- Record Xubuntu release/version and architecture.
- Confirm browser availability and extension support.
- Check Node.js/npm availability if required by the proxy/package.
- Check network connectivity.
- Record CPU, RAM, disk, and available space on the actual Niniy-PC.
- Determine whether the session is temporary or persistent.

Read-only inspection commands may include:

```bash
cat /etc/os-release
uname -a
lsblk -o NAME,SIZE,TYPE,MOUNTPOINTS
free -h
df -h
ip -br link
ip route
command -v chromium chromium-browser google-chrome node npm docker
```

Do not run disk-formatting, partitioning, bootloader, or permanent-install commands as part of this inventory.

### Session 1 — Disposable workspace

Create a dedicated test area, for example:

```text
~/AgChat-Testing/test-a01/
├── workspace/
├── evidence/
├── notes/
└── backups/
```

Initialise Git inside the test workspace and create a baseline commit. Use synthetic files only. Never use a production repository or real secrets as the first fixture.

If Xubuntu is a normal non-persistent Live USB, treat the environment as temporary. Do not assume packages or files will survive reboot.

### Session 2 — MCP SuperAssistant extension and proxy

- Install/load the extension using its documented method.
- Install/configure the local proxy using the version-specific instructions.
- Select either SSE or Streamable HTTP for the initial attempt.
- Confirm the local endpoint and proxy process are running.
- Record versions and configuration with secrets/tokens redacted.

Do not paste authentication tokens into documentation or logs.

### Session 3 — Desktop Commander and handshake

- Install/configure Desktop Commander MCP.
- Set its filesystem scope to the disposable workspace only.
- Connect it to the proxy.
- Verify the MCP handshake and enumerate available tools.
- Start with read-only operations.
- Confirm the browser-side bridge actually passes results back into the originating ChatGPT conversation.

### Session 4 — File operations and Git verification

From the same ChatGPT conversation:
- List workspace files.
- Read a known fixture.
- Create a new file.
- Edit it.
- Read it back.
- Inspect Git status and diff.
- Confirm no operation touched files outside the test workspace.

### Session 5 — Controlled command and end-to-end continuity

Only after the execution boundary is established:
- Run a harmless command or test in the constrained environment.
- Capture stdout, stderr and exit code.
- Test an expected failure.
- Test timeout/disconnection behaviour.
- Make several sequential tool calls in the same chat.
- Check for duplicate execution and stale/incorrect result injection.

Do not enable unrestricted host shell access to make the test pass.

## 9. Conditional candidates after TEST A

These are later experiments, not current installation tasks.

### 9.1 ChatCMD

The reviewed material describes ChatCMD as a possible execution/agent runtime with:
- Streamable HTTP MCP access.
- URL/token authentication.
- Identifiers such as taskId, turnId, agentId and requestId.
- Child-agent lifecycle and nested delegation.
- Durable child reports.
- SQLite-backed state/history.

Potential value:
- Native child/worker lifecycle.
- Delegation and report handling without AgChat building its own coordinator.
- A possible way to test executor/reviewer or parent/child workflows using existing runtime features.

Unresolved validation questions:
1. Can the initial user message and the originating ChatGPT conversation be associated reliably?
2. Does MCP sampling/tool interaction work through the selected bridge?
3. Are task, turn, agent and request identifiers preserved end-to-end?
4. Can a child worker's result be returned to the correct parent conversation?
5. What happens if the browser extension or proxy disconnects?
6. Is an extension-side fallback required for child-worker execution?
7. Are approval and permission boundaries preserved across nested delegation?

Do not adopt ChatCMD unless it works through the bridge without losing conversation identity, access control, or worker routing.

### 9.2 CoS (ChatOnSteroids)

CoS is another candidate because it offers an existing Core and desktop/plugin integration model, and the reviewed material describes Prime/worker chats and local session recording.

Potential value:
- Reuse of existing desktop, plugin, and worker functions.
- A possible local MCP/HTTP integration path.
- Existing interaction/session mechanisms that AgChat should not rebuild.

Unresolved validation questions:
1. Can its local Core MCP endpoint be called safely through the selected local proxy?
2. Is a Secure MCP Tunnel or restricted API key needed only for a remote/native MCP route, or also for the local bridge?
3. Can caller identity and request/chat ownership be preserved?
4. Does worker routing work from the original conversation?
5. Are file and command permissions sufficiently constrained?
6. Does the desktop interaction work on Xubuntu/XFCE, not merely in a GNOME/GTK demonstration environment?
7. What are the RAM, dependency, browser-session, and keyring requirements?

Do not assume that a demonstrated GNOME/GTK workflow proves Xubuntu/XFCE compatibility.

### 9.3 Candidate selection rule

After TEST A, evaluate ChatCMD and CoS against the same evidence-based criteria:
- Same-chat identity/continuity.
- MCP compatibility through the chosen proxy.
- Access control and approval behaviour.
- Local filesystem and command boundaries.
- Worker routing only if required.
- Failure recovery and duplicate-execution prevention.
- Resource use on the actual Niniy-PC.
- Installation and maintenance burden.

Choose an existing runtime only if it solves a demonstrated need. Do not combine runtimes merely because both are available.

## 10. Security and operational model

### 10.1 Threats to consider

- A malformed or over-broad AI-generated command.
- Prompt injection from source files, web pages, issue text, or repository content.
- Accidental writes outside the workspace.
- Secrets copied into logs, prompts, or evidence.
- Duplicate execution after retries or timeouts.
- A local proxy endpoint exposed beyond localhost.
- Over-permissive container mounts or Docker socket access.
- A browser extension or local process being compromised.

### 10.2 Minimum safeguards

- Deny by default.
- Begin read-only.
- Use synthetic fixtures.
- Keep the workspace disposable and Git-controlled.
- Require approval for writes and commands.
- Restrict proxy exposure to localhost unless there is a deliberate, secured reason otherwise.
- Redact tokens and credentials from evidence.
- Use a container with minimal mounts/capabilities for command execution.
- Do not use privileged containers or mount the host home directory.
- Inspect diffs and execution results before accepting a change.
- Record failures honestly; do not mark a test as passed if only part of the loop worked.

### 10.3 Evidence record

Each test run should record:
- Date/time and Xubuntu environment details.
- Browser, extension, proxy, MCP server and package versions.
- Transport and endpoint type (never the secret token).
- Tools discovered.
- Exact test action and expected result.
- Actual result.
- stdout/stderr/exit code for commands.
- Git status/diff.
- Errors, retries, timeouts, and any suspected duplicate execution.
- Pass/fail decision and outstanding risks.

## 11. Explicit exclusions for the current MVP

Do not add these to the current build unless a new requirement is approved:
- OBA / Open Browser Agent.
- Independent reviewer or moderator.
- Custom multi-agent orchestration.
- Personal AI workforce.
- Worker pool implementation.
- Automatic resume as a custom feature.
- Custom task manager/coordinator.
- Custom trace, approval ledger, task timeline, or execution log where native features already exist.
- VM-first deployment.
- Permanent Xubuntu installation or SSD formatting as an assumed part of TEST A.

## 12. Roadmap and gates

### Gate 0 — Environment readiness
**Outcome:** Confirm Xubuntu state and actual Niniy-PC resources.  
**Exit condition:** A stable, understood test environment and disposable workspace plan.

### Gate 1 — TEST A bridge proof
**Outcome:** ChatGPT Go can discover and invoke Desktop Commander through MCP SuperAssistant and the local proxy.  
**Exit condition:** Read, create, edit, controlled execution, Git verification, same-chat continuity, and failure behaviour are evidenced.

### Gate 2 — Runtime comparison
**Outcome:** Evaluate ChatCMD and CoS only against gaps found in TEST A.  
**Exit condition:** One candidate demonstrates a concrete advantage without losing chat identity, access control, or routing.

### Gate 3 — Integrate
**Outcome:** Adopt the minimum existing components needed for the validated workflow.  
**Exit condition:** No duplicated orchestration or safety systems; integration is documented and reproducible.

### Gate 4 — Operate and stabilise
**Outcome:** Run repeatable coding tasks with predictable approvals, evidence, recovery, and resource usage.  
**Exit condition:** Stable workflow on the actual Xubuntu/Niniy-PC setup, with known limitations documented.

## 13. Current decisions register

| ID | Decision | Status |
|---|---|---|
| D-001 | AgChat is the general project/SoT for agentic coding through chat mode. | Accepted |
| D-002 | Reuse existing open-source/native capabilities before building new runtime features. | Accepted |
| D-003 | Preserve one full ChatGPT conversation as the baseline for the initial proof. | Accepted |
| D-004 | One task at a time for the current MVP. | Accepted |
| D-005 | TEST A (MCP SuperAssistant + local proxy + Desktop Commander) is the first implementation test. | Accepted; untested |
| D-006 | ChatCMD and CoS are deferred until TEST A evidence exists. | Accepted |
| D-007 | Do not build a custom reviewer, moderator, coordinator, worker pool, or orchestration framework. | Accepted |
| D-008 | Do not duplicate native trace, approval ledger, task timeline, or execution logs. | Accepted |
| D-009 | OBA is outside AgChat scope. | Accepted |
| D-010 | Host-only is the simplest starting point for read-only/file proof; command execution requires a real boundary. | Accepted |
| D-011 | Container is a possible command-execution boundary; VM is not required for V1. | Accepted |
| D-012 | Do not attribute i7-4710HQ laptop specifications to Niniy-PC. | Constraint |
| D-013 | Do not treat Live USB without persistence as a permanent installation. | Constraint |
| D-014 | Apply practical, risk-proportionate security; do not block the PoC with unnecessary isolation layers. | Accepted standing principle |
| D-015 | The dedicated development PC may run ordinary host-level PoC commands; avoid destructive operations and protect important data. | Accepted |
| D-016 | Do not bypass provider limits, entitlements, or safety policies through multi-chat/multi-agent routing. | Constraint |

## 14. Open questions (not blockers for the first read-only proof)

- Exact Xubuntu release and whether the current boot is Live USB, persistent, or installed.
- Actual CPU/RAM/storage and available disk space on Niniy-PC.
- Browser/extension compatibility in the actual Xubuntu session.
- Which MCP SuperAssistant proxy version and transport will be used.
- Exact Desktop Commander version and tool permissions.
- Whether a constrained container is available and practical on Niniy-PC.
- Whether TEST A can preserve same-chat continuity across multiple sequential calls.
- Whether ChatCMD or CoS can preserve the same identity and security properties if later evaluated.

## 15. Change-control rule

When new evidence appears:
1. Record the test and its environment.
2. Separate observed facts from assumptions.
3. Update the relevant decision status (accepted, validated, rejected, deferred).
4. Explain what evidence justifies changing an accepted decision.
5. Keep historical decisions rather than silently rewriting the project history.

**Current TEST A evidence (27 November 2026):** RDC connection is online; Desktop Commander MCP is connected; `allowedDirectories` is set to `/home/xubuntu/AgChat-Testing/test-a01`; workspace folders `workspace/`, `evidence/`, `notes/`, and `backups/` exist; a synthetic `hello-agchat.md` was created, read, appended, and read back successfully. Git status/diff, command execution, sequential same-chat calls, and failure/retry behaviour remain pending.

**Next action:** Run Git status/diff in the existing synthetic workspace, then test a harmless development command through the already-connected Desktop Commander path. Do not reinstall or repeat the successful setup. ChatCMD/CoS remain deferred until the practical PoC has enough evidence.
