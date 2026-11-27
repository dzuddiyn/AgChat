# TEST A — Xubuntu Execution & Acceptance Plan

**Status:** Planned — not yet executed  
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
- Run one harmless command/test inside a real constrained execution boundary.
- Receive stdout, stderr and exit code.
- Continue with sequential tool calls without duplicate execution or loss of conversation continuity.

A successful extension installation alone is not a pass.

## 2. Architecture under test

`ChatGPT Go → browser extension → local MCP proxy → Desktop Commander MCP → disposable workspace`

Use one MCP transport for the first attempt:
- SSE endpoint discussed: `http://localhost:3006/sse`
- Streamable HTTP endpoint discussed: `http://localhost:3006/mcp`

Confirm the actual endpoint from the installed version's configuration. Do not assume both are active.

## 3. Safety rules

1. Use synthetic files, not production source code or personal documents.
2. Keep the test workspace separate from the home directory's important files.
3. Begin with read-only tools.
4. Limit filesystem access to the test workspace.
5. Do not assume a filesystem allow-list confines terminal commands.
6. Do not enable unrestricted shell execution on the host.
7. Manual approval is not a sandbox.
8. Before running commands, establish a constrained container or equivalent execution boundary.
9. Do not use privileged containers, broad home mounts, or host Docker socket access.
10. Redact tokens, passwords, and private URLs from logs and evidence.
11. Do not run sudo, partition, format, reinstall, or bootloader commands as part of this test.
12. If using a non-persistent Live USB, treat changes to installed packages/configuration as temporary and potentially lost on reboot.

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

## 5. Phase 1 — Disposable workspace

Proposed layout:

```text
~/AgChat-Testing/test-a01/
├── workspace/
├── evidence/
├── notes/
└── backups/
```

Inside `workspace/`:
- Initialise Git.
- Create a baseline commit.
- Add a small synthetic fixture such as `hello.txt` with a known value.
- Record the initial Git status.

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

## 9. Phase 5 — Controlled command/test

This phase is blocked until a real command-execution boundary is selected and configured.

Recommended direction: a non-privileged container with only the disposable workspace mounted. Avoid privileged mode, host socket mounts, and broad host directory mounts.

Run only a harmless test, for example printing a known marker or running a tiny local fixture test.

Capture:
- Command identifier/action.
- stdout.
- stderr.
- Exit code.
- Duration if available.
- Git status/diff after execution.

Also test one expected failure, such as a harmless invalid test target, to confirm non-zero exit codes and error output are returned honestly.

**Exit condition:** Command executes only inside the constrained environment and its complete result returns to the original chat.

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

**FAIL** if the bridge cannot return tool results reliably, executes duplicate writes/commands, or cannot maintain a safe workspace boundary.

A partial pass is useful evidence. Do not label it a full pass.

## 13. What happens after TEST A

Only after the evidence is reviewed:
1. Identify the actual gaps.
2. Evaluate ChatCMD against those gaps.
3. Evaluate CoS against those gaps.
4. Compare chat identity, access control, worker routing, recovery, and resource use.
5. Adopt an existing runtime only if it demonstrably adds value.

Until then, do not install ChatCMD or CoS and do not build a custom orchestration framework.
