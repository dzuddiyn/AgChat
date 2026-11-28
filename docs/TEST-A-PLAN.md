# TEST A — Xubuntu Execution & Acceptance Plan

**Status:** Session 0 — Xubuntu readiness: PASS. Session 1 — Disposable workspace: PASS. Session 2 — MCP SuperAssistant extension and proxy: NEXT.  
**Priority:** First implementation gate for AgChat  
**Baseline:** ChatGPT Go, one original conversation, MCP SuperAssistant, local proxy, Desktop Commander  
**Target host:** Niniy-PC Mini PC running/testing Xubuntu  
**Workspace:** Disposable synthetic test workspace only

This plan is intentionally narrow. It tests the local tool bridge before adding ChatCMD, CoS, worker routing, or any custom orchestration.

## 1. Objective

Prove that ChatGPT Go, from the same conversation, can:
- Discover local MCP tools through MCP SuperAssistant.
- Read, create, and edit files in a scoped local workspace.
- Verify changes through Git.
- Run a harmless development command/test through the connected local tool path.
- Receive stdout, stderr and exit code.
- Continue with sequential tool calls without duplicate execution or loss of conversation continuity.

A successful extension installation alone is not a pass.

## 2. Architecture under test

`ChatGPT Go → browser extension → local MCP proxy → Desktop Commander MCP → disposable workspace`

Use one MCP transport for the first attempt:
- SSE endpoint discussed: `http://localhost:3006/sse`
- Streamable HTTP endpoint discussed: `http://localhost:3006/mcp`

Confirm the actual endpoint from the installed version's configuration. Do not assume both are active.

## 3. Practical operating rules

This is a PoC on a PC dedicated to development. The priority is to get a working workflow into use, not to build a production security environment.

1. Use the existing Xubuntu session and connected tools; do not repeat successful setup.
2. Use the existing synthetic workspace for the initial checks.
3. Filesystem allow-listing is useful, but it does not confine terminal commands.
4. Ordinary, understood, non-destructive host commands are acceptable for this PoC; a container/VM is not a prerequisite.
5. Avoid destructive disk/system operations (formatting, partitioning, bootloader changes, broad deletion) and do not expose credentials or important personal/production data.
6. Inspect Git status/diff after file-changing operations.
7. Keep tokens, passwords, and private credentials out of logs and evidence.
8. If this remains a non-persistent Live USB, remember that installed packages/configuration may be lost on reboot.
9. Add stronger isolation only if a concrete risk or later workload justifies it.

## 4. Phase 0 — Xubuntu readiness inventory

First establish what is actually running. Do not install or modify anything yet.

Read-only inspection commands:

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

Record:
- Xubuntu release/version and architecture.
- Live USB, persistent Live USB, or installed system.
- Browser name/version.
- Node.js/npm availability and versions, if present.
- Network connectivity.
- Actual Niniy-PC CPU/RAM/storage and free space.
- Whether the current session survives reboot.

**Exit condition:** Environment and persistence status are known. If this is a normal Live USB without persistence, proceed only with a temporary proof and avoid unnecessary package installation.


**Session 0 result: PASS — installed Xubuntu environment verified on 28 November 2026.**

- OS: Ubuntu 26.04.1 LTS (Resolute Raccoon), Xfce 4.20.4; session identifies as Xubuntu.
- Boot/install mode: installed system; root filesystem is on internal SSD `/dev/sda2`.
- Internal SSD: 119.2 GB disk; root filesystem 116 GB total, 94 GB available at inspection.
- Memory: 11 GiB RAM; 4 GiB swap.
- CPU architecture: x86_64.
- Browser: Firefox at `/usr/bin/firefox`.
- Toolchain: Git 2.53.0, Node.js v24.22.3, npm 11.19.1, Python 3.14.4.
- Network interface `eno1` is up with an assigned IPv4 address.
- Desktop Commander Remote device `dzu-miniPC` is online.
- Additional storage detected: 1 TB internal HDD, 2 TB external drive, and 58.6 GB USB drive.

The old Live USB workspace is not present in the new installation. Session 1's result is retained in the repository; do not repeat its coding-loop tests. A fresh disposable workspace may be recreated when needed for bridge validation.

## 5. Phase 1 — Disposable workspace

**Current result: PASS — Session 1 completed.** The disposable workspace was created at `/home/xubuntu/AgChat-Testing/test-a01/` with `workspace/`, `evidence/`, `notes/`, and `backups/`. Desktop Commander `allowedDirectories` was set to this test directory and read back successfully. A synthetic `hello-agchat.md` fixture was created, read, appended, and read back. Git baseline commit `e7b3fd2` was created. A Python monthly-budget calculator (`budget.py`) and 8 unit tests were added; all 8 tests passed. A controlled defect (addition instead of subtraction) was injected, detected by the tests, repaired, and verified with all 8 tests passing again. The correct output returned RM2,250. Git staging and diff checks passed. This validates the **local RDC/Desktop Commander coding loop and failure-recovery subtest**. It does not validate the MCP SuperAssistant browser-extension/proxy bridge.

Do not recreate this workspace or repeat these successful file operations.

Proposed layout:

```text
~/AgChat-Testing/test-a01/
├── workspace/
├── evidence/
├── notes/
└── backups/
```

Inside `workspace/`:
- Check whether Git is already initialised; do not overwrite existing repository metadata.
- If not initialised, initialise Git in this disposable workspace and create a baseline commit if Git identity is already configured.
- Keep the existing synthetic `hello-agchat.md` as the fixture.
- Record Git status and diff.

Do not place secrets or real user data in this workspace.

**Exit condition:** Clean baseline commit exists and workspace path is confirmed.

## 6. Phase 2 — MCP SuperAssistant extension and local proxy

- Use the official instructions for the selected extension/proxy version.
- Load the extension using the supported browser installation method.
- Start/configure the local proxy.
- Select SSE or Streamable HTTP for this test, not both simultaneously.
- Confirm the endpoint listens locally.
- Record versions and non-secret configuration.

**Exit condition:** Extension can communicate with the local proxy and the proxy reports a healthy connection.

## 7. Phase 3 — Desktop Commander and MCP handshake

- Install/configure Desktop Commander according to its version-specific documentation.
- Scope filesystem access to the disposable workspace only.
- Connect Desktop Commander to the proxy.
- Complete MCP handshake and enumerate tools.
- Start with read-only operations.
- Verify that the tool result is injected into the original ChatGPT conversation.

**Exit condition:** Tool discovery succeeds and a read-only call returns the expected result in the same conversation.

## 8. Phase 4 — Filesystem and Git test

Perform these operations sequentially from the original conversation:

| Test | Action | Expected evidence |
|---|---|---|
| A-READ | Read the known fixture | Exact expected contents |
| A-CREATE | Create a new file | File exists in workspace |
| A-EDIT | Change a known value | Read-back shows changed value |
| A-STATUS | Run Git status | Only expected files are modified/untracked |
| A-DIFF | Inspect Git diff | Diff matches requested change |
| A-BOUNDARY | Attempt a harmless path check outside workspace | Operation is denied or demonstrably unavailable |

Do not deliberately write outside the workspace to test the boundary. A denied read/list check is sufficient if the tool exposes a safe way to test it.

**Exit condition:** File operations work, changes are verifiable, and workspace scoping is demonstrated.

## 9. Phase 5 — Practical command/test

No container or VM is required for this PoC. Use the connected Desktop Commander command tool on the dedicated development PC for ordinary, non-destructive development commands.

Start with a harmless check such as printing a marker, checking the Git version/status, or running a tiny local fixture test. Do not run destructive disk/system commands or broad deletion.

Capture:
- Command identifier/action.
- stdout.
- stderr.
- Exit code.
- Duration if available.
- Git status/diff after execution.

Also test one expected failure, such as a harmless invalid test target, to confirm non-zero exit codes and error output are returned honestly.

**Exit condition:** A harmless command completes through the connected tool path and its result (stdout, stderr, exit code where exposed) returns to the original chat.

## 10. Phase 6 — Same-chat continuity and failure behaviour

Within the same ChatGPT conversation:
1. Ask for a file read.
2. Ask for a file creation.
3. Ask for a file edit.
4. Ask for Git verification.
5. Ask for the controlled test.
6. Ask the model to interpret the evidence and propose the next action.

Test:
- Proxy disconnect/reconnect.
- A harmless timeout or failed operation.
- A repeated request where the first call may already have completed.
- Whether one request causes more than one execution.
- Whether the returned result is associated with the correct tool request.

**Exit condition:** Sequential calls remain in the original conversation, failures are explicit, and duplicate execution is not observed.

## 11. Evidence template

For each test, record:

```text
Test ID:
Date/time:
Xubuntu release:
Boot/install mode:
Host: Niniy-PC
Browser/version:
SuperAssistant version:
Proxy version:
MCP transport:
Desktop Commander version:
Workspace path:
Action:
Expected result:
Actual result:
stdout:
stderr:
exit code:
Git status/diff:
Retries/timeouts:
Duplicate execution observed?:
Pass / Fail / Blocked:
Notes:
```

Do not include authentication tokens or secrets.

## 12. Overall pass/fail criteria

**PASS** only when all required capabilities are demonstrated:
- MCP connection and discovery.
- Read, create and edit.
- Git status and diff verification.
- Controlled command/test with stdout, stderr and exit code.
- Same-chat sequential continuity.
- No duplicate execution.
- Honest failure behaviour.
- Evidence recorded.

**PARTIAL** if filesystem operations work but command execution or same-chat continuity remains unproven.

**FAIL** if the bridge cannot return tool results reliably, executes duplicate writes/commands, or causes unintended destructive changes. Do not fail the PoC merely because no container/VM is present.

A partial pass is useful evidence. Do not label it a full pass.

## 13. Current progress and immediate next actions

| Item | Status | Evidence / next action |
|---|---|---|
| RDC connection | PASS | Device `xubuntu` is online. |
| Desktop Commander MCP connection | PASS | Tool calls succeeded. |
| Filesystem scope | PASS | `allowedDirectories` points to `/home/xubuntu/AgChat-Testing/test-a01`. |
| Workspace layout | PASS | `workspace/`, `evidence/`, `notes/`, `backups/` exist. |
| File create/read/edit | PASS | `hello-agchat.md` created, read, appended and read back. |
| Git status/diff | PASS | Git 2.53.0 installed; repository initialised in test workspace; staged diff inspected; baseline commit `e7b3fd2`; final status clean. |
| Harmless command execution | PASS | `printf` and Python smoke test both executed via Desktop Commander; exit code 0. |
| Same-chat sequential calls | PENDING | Continue several tool calls in this same conversation. |
| Failure/retry/duplicate behaviour | PENDING | Test a safe failure and verify no duplicate write. |

**Latest results:** Git 2.53.0 installed successfully through RDC. Git was initialised only inside `/home/xubuntu/AgChat-Testing/test-a01/workspace`; `hello-agchat.md` was staged and its cached diff inspected. Baseline commit: `e7b3fd2 test: add synthetic AgChat fixture`. A repository-local test identity was used, without changing global Git identity. Final `git status --short` was clean. A harmless shell `printf` and Python smoke test both returned exit code 0 through Desktop Commander.

**Next:** Stop work on the temporary Live USB. Back up Windows data, install Xubuntu to the SSD, and complete the basic environment setup. Then resume with **Session 2 — MCP SuperAssistant extension and proxy**. Do not repeat Session 1; its disposable-workspace coding-loop subtests are recorded as PASS. The browser-extension → local proxy → Desktop Commander end-to-end bridge remains unverified.

## 14. What happens after TEST A

Only after the evidence is reviewed:
1. Identify the actual gaps.
2. Evaluate ChatCMD against those gaps.
3. Evaluate CoS against those gaps.
4. Compare chat identity, access control, worker routing, recovery, and resource use.
5. Adopt an existing runtime only if it demonstrably adds value.

Until then, do not install ChatCMD or CoS and do not build a custom orchestration framework.
