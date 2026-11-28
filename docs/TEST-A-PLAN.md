# TEST A — Xubuntu Execution & Acceptance Plan

**Status:** **PASS — agreed V1 PoC scope (28 November 2026), with documented limitations.** Session 0/1/2 PASS; Test 43 E2E PASS; Test 44 controlled success/failure PASS and fixture-level duplicate guard PASS (duplicate exit-code telemetry incomplete); Test 45 filesystem allow-list boundary PASS; Test 46 practical coding loop PASS (four unit tests, exit 0, read-back verified). Generic transport-level exactly-once and proxy reconnect recovery are not claimed or required for this V1 PoC. Shell/OS isolation is not provided by the filesystem allow-list; reviewed, non-destructive host commands require manual approval.  
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

**Session 2 result: PASS.** The MCP SuperAssistant extension reported `Connected` and `30 tools`; its Desktop Commander group showed `25/25` enabled and Browser showed `5/5`. The local proxy was configured for SSE at `http://localhost:3006/sse` (proxy listener previously verified on `127.0.0.1:3006`). `get_tool_schema` returned the actual schema for `desktop-commander.write_file` and `desktop-commander.read_file`. A read-only `list_directory` call also returned the actual contents of `/home/dzu/AgChat-Testing/session-2`. The write and read calls below then confirmed the round trip.

- Install/configure Desktop Commander according to its version-specific documentation.
- Connect Desktop Commander to the proxy.
- Complete MCP handshake and enumerate tools.
- Verify that tool results are returned to the originating ChatGPT conversation.

**Exit condition: PASS for MCP connection, tool discovery, and read-only result return.** This does not yet prove the complete TEST A acceptance criteria.

## 8. Phase 4 — Filesystem and Git test

**Partial result — Session 2:** `A-CREATE`, `A-READ`, `A-EDIT`, and edited read-back passed in `/home/dzu/AgChat-Testing/session-2/`. The file was created with `AgChat TEST A Session 2 - WRITE PASS`, read back exactly, overwritten with the two lines `AgChat TEST A Session 2 - EDIT PASS` and `MCP file edit and read-back verification.`, then read back exactly. Git status/diff initially failed because the disposable Session 2 folder had no `.git` directory, although Git 2.53.0 was installed. No `user.name`/`user.email` identity was configured either. Resolved by running `git init -b main` in this disposable workspace and setting repository-local identity (`AgChat Test`, `agchat-test@localhost`) without changing global configuration. Committed the existing test fixture as baseline `0a3027a`, appended a harmless verification line, and reran the checks: `git status --short` exit 0 showed the expected modified fixture plus two untracked test/config files; `git diff -- agchat-write-test.txt` exit 0 showed the exact appended line. Git verification: PASS. Boundary checks remain pending. The new workspace path is `/home/dzu/AgChat-Testing/session-2/`, not the old Live USB workspace.

Perform the remaining operations sequentially from the original conversation:

| Test | Action | Expected evidence |
|---|---|---|
| A-READ | Read the known fixture | Exact expected contents |
| A-CREATE | Create a new file | File exists in workspace |
| A-EDIT | Change a known value | Read-back shows changed value — PASS |
| A-STATUS | Run Git status | PASS after local `git init`; exit 0 and expected modified/untracked files shown. |
| A-DIFF | Inspect Git diff | PASS; exit 0 and exact fixture change shown. |
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
| Filesystem scope | PASS | Current Session 2 allow-list is `/home/dzu/AgChat-Testing/session-2`; Test 45 denied out-of-scope `/home/dzu/Documents` via the actual SuperAssistant proxy route. |
| Workspace layout | PASS | `workspace/`, `evidence/`, `notes/`, `backups/` exist. |
| Session 1 local file create/read/edit | PASS | `hello-agchat.md` created, read, appended and read back through direct RDC/Desktop Commander. |\n| Session 2 MCP file create/read | PASS | `agchat-write-test.txt` created and read back with exact content through MCP SuperAssistant → proxy → Desktop Commander. |\n| Session 2 MCP file edit | PASS | Two-line rewrite and exact read-back verified through MCP SuperAssistant → proxy → Desktop Commander. |
| Git status/diff (Session 1) | PASS | Git evidence belongs to Session 1 workspace `/home/xubuntu/AgChat-Testing/test-a01/workspace`. |\n| Git status/diff (Session 2) | PASS | Separate local repository initialised in `/home/dzu/AgChat-Testing/session-2`; baseline `0a3027a`; status/diff exit 0. Repository-local test identity only. |
| Harmless command execution | PASS | `printf` and Python smoke test both executed via Desktop Commander; exit code 0. |
| Same-chat sequential calls | PASS (baseline) | Test 43 and Test 44 multi-step calls completed in the same originating ChatGPT conversation. Cross-refresh/session recovery is not a V1 acceptance requirement. |
| Failure/retry/duplicate behaviour | PASS for V1 fixture guard; telemetry PARTIAL | Test 44 expected failure exit 7 and success exit 0 PASS; duplicate output `DUPLICATE_BLOCKED` and state count 1 confirmed. Exit 17 was unavailable after session expiry; no rerun. Generic proxy transport deduplication is not claimed or required. |

**Session 1 historical results:** Git 2.53.0 installed successfully through RDC. Git was initialised only inside `/home/xubuntu/AgChat-Testing/test-a01/workspace`; `hello-agchat.md` was staged and its cached diff inspected. Baseline commit: `e7b3fd2 test: add synthetic AgChat fixture`. A repository-local test identity was used, without changing global Git identity. Final `git status --short` was clean. A harmless shell `printf` and Python smoke test both returned exit code 0 through Desktop Commander.

**Session 2 environment and evidence (28 November 2026):** Installed Xubuntu on `dzu-miniPC`, home directory `/home/dzu`. Chrome Stable was used with MCP SuperAssistant `0.7.3`. The extension displayed `Connected · 30 tools` (Desktop Commander `25/25`, Browser `5/5`). Proxy transport: SSE; configured endpoint `http://localhost:3006/sse`; proxy listener had been verified on `127.0.0.1:3006`. The extension returned the actual schemas for `desktop-commander.write_file` and `desktop-commander.read_file`. A read-only directory listing returned `/home/dzu/AgChat-Testing/session-2` with `config.json`. The write tool created `agchat-write-test.txt` and returned `Successfully wrote to /home/dzu/AgChat-Testing/session-2/agchat-write-test.txt (1 lines)`. The read tool then returned `[Reading 1 lines from start (total: 1 lines, 0 remaining)]` followed by the exact expected content `AgChat TEST A Session 2 - WRITE PASS`. No other file was intentionally changed in this test.

**Session 2 follow-up (direct RDC):** A harmless Python command emitted separate stdout/stderr markers and returned shell exit 0 (137 ms). A controlled expected failure emitted both streams and child exit 7. An exclusive-create idempotency fixture returned `EXECUTED_ONCE` on first invocation and `DUPLICATE_BLOCKED` on second; read-back confirmed `EXECUTION_COUNT=1`. This proves the fixture's guard, not automatic transport-level deduplication. Desktop Commander config was changed from `allowedDirectories: []` (unrestricted filesystem access) to `["/home/dzu/AgChat-Testing/session-2"]`. In-scope listing succeeded; listing `/home/dzu/Documents` was denied with `Path not allowed`. This allow-list constrains Desktop Commander filesystem tools, not arbitrary shell commands; it is not an OS sandbox.\n\n**Next:** Validate controlled command execution, expected failure reporting and a no-duplicate retry through the already-working MCP SuperAssistant extension/proxy path. Test 43 has already proven the ChatGPT → proxy → Orchestrator → Desktop Commander route; do not replay it. Do not repeat Session 1, reinstall the bridge, or recreate the browser profile.

## 14. What happens after TEST A

TEST A has passed for the agreed V1 PoC scope, with limitations recorded in the roadmap.

1. Use the validated ChatGPT + SuperAssistant + Desktop Commander workflow for the next concrete AgChat implementation task.
2. Keep ChatCMD and CoS deferred unless a real task reveals a specific missing capability.
3. If a gap is demonstrated, compare only the candidate(s) that address that gap against chat identity, permissions, resource use and maintenance burden.
4. Do not add custom orchestration, worker routing, or a container/VM by default.


## 15. Test 43 — SuperAssistant proxy + Orchestrator E2E

**Result: PASS (28 November 2026).** The originating ChatGPT conversation invoked `agchat-orchestrator.agchat_reconcile_test43` through the isolated MCP SuperAssistant extension and local SSE proxy. The proxy log confirms the `tools/call`; the Orchestrator then called Desktop Commander `read_file` and verified the existing proof file. The operation ledger ended in `SUCCEEDED` with a `RECONCILED_SUCCEEDED` event. The fixture was not rerun during reconciliation.

- Proxy endpoint: `http://localhost:3006/sse`.
- Proxy config: `config-test43.json` (Desktop Commander + AgChat Orchestrator).
- Tool counts: Desktop Commander 25; Orchestrator 5.
- Proof: `/home/dzu/AgChat-Testing/session-2/agchat-test43-proxy-e2e-proof.txt`.
- SHA-256: `172b5e80f247d214a99166664f4738892ea2894ebc0e8475a2865e3e82c951eb`.
- Auto Execute: OFF; manual Run only.

The initial execution was marked FAILED due to an Orchestrator response-parser mismatch, although Desktop Commander had created the proof. The reconciliation tool independently read the existing marker through Desktop Commander and corrected the ledger without repeating the side effect. Full detail: [TEST 43 result](TEST-43-PROXY-ORCHESTRATOR-RESULTS.md).

**Boundary:** This validates the exact tested route and fixed fixture. It does not prove transport-level exactly-once semantics or generic duplicate suppression. Do not upgrade overall TEST A to PASS until the remaining acceptance items are independently closed.


## 16. Browser operating constraint and next checkpoint

**Browser constraint (mandatory):** use Google Chrome with the Rdeeen profile only. The working profile clone is `/home/dzu/AgChat-Testing/session-2/chrome-user-profile-clone`, signed in as `Rdeeen@gmail.com`. Do not launch, open, focus or use Vivaldi or Firefox. Reuse the existing Chrome process and CDP endpoint `http://127.0.0.1:9222`; do not start another Chrome profile or a conflicting CDP instance. See repository `AGENTS.md` and [Development Method](AGCHAT-WORKING-METHOD.md).

**Verified checkpoint:** Test 43 is PASS. The SSE proxy is listening on port 3006; the Orchestrator ledger is SUCCEEDED; the proof hash is recorded in [Test 43 results](TEST-43-PROXY-ORCHESTRATOR-RESULTS.md). Do not repeat its setup, fixture, approval or reconciliation.

**Checkpoint status:** Tests 43–45 have now covered the proxy route, controlled success/failure, same-chat sequential calls and filesystem allow-list denial. Test 44 duplicate process exit 17 remains PARTIAL because the original process session expired; do not rerun the fixture.

**Next practical test:** complete one small, non-destructive coding task in the existing `/home/dzu/AgChat-Testing/session-2/` workspace through the current Rdeeen Chrome → SuperAssistant → proxy → Desktop Commander route. Inspect the resulting file and Git diff before accepting it. Do not repeat setup or passing tests.

**Acceptance boundary:** generic transport-level exactly-once delivery and cross-refresh recovery are not V1 requirements. Use idempotent actions where practical and inspect the original process/state before retrying an ambiguous operation. Keep overall TEST A PARTIAL until the practical coding loop is demonstrated.

## 17. Test 44 — controlled command execution, failure and duplicate guard

**Result: PARTIAL (28 November 2026).** See [Test 44 detailed report](TEST-44-COMMAND-FAILURE-DUPLICATE-RESULTS.md).

- Expected failure through SuperAssistant proxy: stdout/stderr markers and exit code 7 verified using follow-up `read_process_output` (PASS).
- Success through SuperAssistant proxy: `CREATED`, 67-byte state file, stderr marker and exit code 0 verified (PASS).
- Duplicate attempt through SuperAssistant proxy: `DUPLICATE_BLOCKED` and no-second-side-effect message received; independent read-back confirms `execution_count=1` and unchanged SHA-256. Exit code 17 not independently captured (PARTIAL).
- All tool cards were run manually; Auto Execute remained OFF.
- Test 43 setup and fixture were not repeated or modified.

**Final disposition:** the original duplicate process session expired and a later read returned `No session found for PID 126708`. Do not rerun the fixture. `DUPLICATE_BLOCKED`, state count 1 and unchanged SHA-256 establish the fixture-level no-second-side-effect guard; exit-code telemetry remains incomplete. Generic transport-level exactly-once is not claimed or required for V1.


## 18. Test 45 — workspace filesystem boundary

**Result: PASS for Desktop Commander filesystem tools (28 November 2026).** See [Test 45 detailed report](TEST-45-WORKSPACE-BOUNDARY-RESULTS.md).

A single read-only `desktop-commander.list_directory` request for `/home/dzu/Documents` through the originating ChatGPT conversation and SuperAssistant proxy was denied with `Path not allowed`. The error explicitly identified the sole allowed directory: `/home/dzu/AgChat-Testing/session-2`. No out-of-scope data was returned and no side effect occurred.

**Boundary qualification:** This verifies the filesystem allow-list only. It does not confine arbitrary shell/process execution. For this dedicated-PC PoC, host-level non-destructive commands remain acceptable with manual approval and sensible safeguards; container/VM isolation is optional, not a V1 prerequisite.

## 19. Refined remaining acceptance items

1. **Duplicate/retry evidence:** Test 44 duplicate guard is confirmed at fixture level (state remains count 1), but exit 17 could not be recovered because the original Desktop Commander process session no longer exists. Do not rerun the side-effecting fixture. Generic transport-level exactly-once delivery is not claimed.
2. **Failure recovery:** Proxy disconnect/reconnect and ambiguous timeout recovery remain untested. If tested, inspect the original request/process/side effect before any retry.
3. **Same-chat baseline:** PASS for multiple sequential calls in the originating conversation (Tests 43–45). Cross-refresh/session recovery is outside the current V1 gate unless a real need arises.

Overall TEST A remains PARTIAL until the remaining failure-recovery and duplicate/retry acceptance is either evidenced or explicitly scoped as a documented V1 limitation. Do not turn generic exactly-once transport semantics into an unbounded requirement.


## 20. Test 46 — practical coding loop

**Result: PASS (28 November 2026).** See [Test 46 detailed report](TEST-46-PRACTICAL-CODING-LOOP-RESULTS.md).

Through the originating ChatGPT conversation and existing SuperAssistant/proxy route:
- Created `agchat_coding_task_01.py` and `test_agchat_coding_task_01.py`.
- Ran `python3 -m unittest -v test_agchat_coding_task_01`: all four tests passed.
- Follow-up `read_process_output` confirmed exit code 0.
- `read_multiple_files` read back both complete files.
- Targeted Git status shows only the two Test 46 files as untracked; no staging or commit was performed.
- SHA-256 hashes are recorded in the detailed report.

**TEST A acceptance decision:** PASS for the agreed V1 PoC scope, with limitations documented in the roadmap. This is not production-grade shell isolation, generic transport exactly-once delivery, or proxy reconnect recovery.
