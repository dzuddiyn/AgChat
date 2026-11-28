# TEST A — Session 2 Results: MCP SuperAssistant Bridge

**Result: PASS for MCP bridge connection, tool discovery, file create/read, and file edit/read-back.**  
**Overall TEST A: PARTIAL — remaining acceptance checks are still open.**  
**Date:** 28 November 2026  
**Host:** `dzu-miniPC`, installed Xubuntu 26.04.1 LTS  
**Home directory:** `/home/dzu`  
**Workspace:** `/home/dzu/AgChat-Testing/session-2/`

## 1. Purpose

Validate the browser-extension-to-local-execution bridge:

`ChatGPT conversation → MCP SuperAssistant (Chrome) → local MCP proxy → Desktop Commander → Xubuntu`

This session continues the work after Session 1. It does not repeat the Session 1 local coding-loop tests.

## 2. Environment and connection evidence

- Browser: Google Chrome Stable.
- MCP SuperAssistant extension: version `0.7.3`.
- Extension status: `Connected`.
- Extension tool count: `30 tools`.
- Tool groups shown by the extension: Desktop Commander `25/25`; Browser `5/5`.
- Proxy transport: Server-Sent Events (SSE).
- Configured SSE endpoint: `http://localhost:3006/sse`.
- Proxy listener was previously verified on `127.0.0.1:3006`.
- Proxy startup log showed Desktop Commander connected, 25 tools and 2 resources loaded, initialization successful, and listener active.

The 30 extension tools comprise 25 Desktop Commander tools and 5 Browser tools. The 5 Browser tools are not part of the Desktop Commander tool count.

## 3. Tool discovery and schema checks

The extension successfully surfaced and ran `get_tool_schema` for:

- `desktop-commander.write_file`
- `desktop-commander.read_file`

The returned schemas showed:
- `write_file` requires `path` and `content`; `mode` supports `rewrite` and `append`.
- `read_file` requires `path`; `offset` and `length` support line-based reading.

These were actual tool results displayed by MCP SuperAssistant, not inferred from documentation.

## 4. Read-only directory check

A `list_directory` call returned the contents of:

`/home/dzu/AgChat-Testing/session-2`

Observed entry:
- `config.json`

This confirmed a read-only result could travel back into the originating ChatGPT conversation.

## 5. Write test — PASS

**Tool:** `desktop-commander.write_file`  
**Path:** `/home/dzu/AgChat-Testing/session-2/agchat-write-test.txt`  
**Mode:** `rewrite`  
**Content:**

```text
AgChat TEST A Session 2 - WRITE PASS
```

**Actual tool result:**

```text
Successfully wrote to /home/dzu/AgChat-Testing/session-2/agchat-write-test.txt (1 lines)
```

Result: **PASS**. The tool reported a successful one-line write to the requested path.

## 6. Read-back verification — PASS

**Tool:** `desktop-commander.read_file`  
**Path:** `/home/dzu/AgChat-Testing/session-2/agchat-write-test.txt`  
**Offset:** `0`  
**Length:** `10`

**Actual returned output:**

```text
[Reading 1 lines from start (total: 1 lines, 0 remaining)]

AgChat TEST A Session 2 - WRITE PASS
```

Result: **PASS**. The read-back content exactly matches the content sent to `write_file`.

## 7. File edit test — PASS

**Tool:** `desktop-commander.write_file` with rewrite mode  
**Path:** `/home/dzu/AgChat-Testing/session-2/agchat-write-test.txt`

The file was overwritten with:

```text
AgChat TEST A Session 2 - EDIT PASS
MCP file edit and read-back verification.
```

The tool reported a successful two-line write. A subsequent `read_file` returned both exact lines. Result: **PASS**.

## 8. Git verification — PASS after root-cause correction

### Root cause
The Session 2 workspace was a plain directory, not a Git repository. Git 2.53.0 was installed and working, but no `.git` directory existed. In addition, neither `user.name` nor `user.email` was configured at system/global scope. The first Git check therefore failed with exit codes 128 (`git status`) and 129 (`git diff`). This was a workspace setup issue, not an MCP, Desktop Commander, or Git installation failure.

### Resolution applied
Because this is a disposable synthetic test workspace, a local Git repository was initialised here; the AgChat source repository was not cloned or modified. Repository-local identity was configured only (no global Git settings changed):

```bash
cd /home/dzu/AgChat-Testing/session-2
git init -b main
git config --local user.name 'AgChat Test'
git config --local user.email 'agchat-test@localhost'
git add -- agchat-write-test.txt
git commit -m 'test: establish Session 2 Git baseline'
```

Baseline commit:
```text
0a3027a test: establish Session 2 Git baseline
```

A harmless line was then appended to the tracked fixture and both checks were rerun. Actual output:

```text
=== GIT STATUS ===
 M agchat-write-test.txt
?? agchat-single-execution-marker.txt
?? config.json
STATUS_EXIT_CODE=0

=== GIT DIFF ===
diff --git a/agchat-write-test.txt b/agchat-write-test.txt
index 0cfa39d..0136d8d 100644
--- a/agchat-write-test.txt
+++ b/agchat-write-test.txt
@@ -1,2 +1,3 @@
 AgChat TEST A Session 2 - EDIT PASS
-MCP file edit and read-back verification.
\\ No newline at end of file
+MCP file edit and read-back verification.
+Git verification fixture change.
DIFF_EXIT_CODE=0
```

The tracked fixture change is visible exactly as expected. The two untracked files (`agchat-single-execution-marker.txt` and `config.json`) were deliberately left untracked and were not committed. A final `git diff --check` returned exit code 0.

Result: **PASS**. Git status and diff are now operational and verified in Session 2.

## 9. Acceptance results

| Check | Result | Evidence |
|---|---|---|
| Extension ↔ proxy connection | PASS | Extension displayed Connected. |
| Tool discovery | PASS | 30 tools displayed; Desktop Commander 25/25. |
| Schema retrieval | PASS | Actual write_file and read_file schemas returned. |
| Read-only directory listing | PASS | Actual session-2 directory entry returned. |
| File creation through MCP | PASS | write_file returned successful one-line write. |
| File read-back through MCP | PASS | Exact content returned. |
| File edit through MCP | PASS | Two-line rewrite and exact read-back verified. |
| ChatGPT UI → SuperAssistant → proxy → Desktop Commander read call | PASS | Call #9 `desktop-commander.read_file`; result success; 170 chars; 48 ms; exact three-line file content returned. |
| Git status and diff | PASS | Local repository initialised; baseline `0a3027a`; status and diff exit 0; final `git diff --check` exit 0. |
| Harmless command/test | PASS via MCP bridge | `desktop-commander.start_process`; stdout/stderr markers captured; PID 61954. |
| Safe expected failure | PASS via direct RDC; bridge controlled exit-7 test PASS | Direct RDC and MCP bridge captured stdout/stderr; bridge PID 61954 exit code 7 verified by read_process_output. |
| Workspace boundary | PASS for filesystem tools | `allowedDirectories` set to session-2; in-scope listing passed; `/home/dzu/Documents` denied. Shell commands are not confined by this allow-list. |
| Auto Insert + Auto Submit (original v0.7.3) | FAIL | On the original extension, the localized ChatGPT send button (`aria-label="Hantar"`) was not included in the submit selector; result was inserted but not submitted. |


### MCP bridge process execution — PASS

**Tool:** `desktop-commander.start_process`  
**Call ID:** `11`  
**PID:** `61954`  
**Working directory:** `/home/dzu/AgChat-Testing/session-2/`

Command (synthetic, no file writes):

```bash
python3 -c 'import sys; print("STDOUT_MARKER"); print("STDERR_MARKER", file=sys.stderr); sys.exit(7)'
```

The extension returned both output markers and PID 61954. The process was then inspected through a second MCP call, `desktop-commander.read_process_output` (call ID 13), without rerunning it.

Actual verified result:

```text
STDOUT: STDOUT_MARKER
STDERR: STDERR_MARKER
Exit code: 7
Runtime: 0.13s
Process: Completed
PID: 61954
```

Result: **PASS** for bridge command execution, stdout/stderr separation, and exact non-zero exit-code read-back. No file was modified by this test.

## 10. Command, expected failure, duplicate guard, and boundary checks (direct RDC)

These checks were run through the Desktop Commander Remote Desktop Commander (RDC) tool on `dzu-miniPC`. They validate the host command path, but **do not yet prove command execution through the MCP SuperAssistant browser-extension/proxy bridge**.

### Harmless command — PASS (direct RDC)

Command executed a Python one-liner that wrote a marker to stdout and a separate marker to stderr, then exited 0.

Actual output:
```text
STDERR_CHANNEL_PASS
AGCHAT_SESSION2_COMMAND_PASS
SHELL_EXIT_CODE=0
```

Timing: 137 ms. Tool timing reported separate stderr and stdout events.

### Expected failure — PASS (direct RDC)

A controlled Python command printed to both output channels and intentionally exited with code 7. The wrapper captured the child exit code without making the outer RDC call itself fail.

Actual output:
```text
EXPECTED_FAILURE_STDERR
EXPECTED_FAILURE_STDOUT
CHILD_EXIT_CODE=7
```

Timing: 119 ms. The expected non-zero child exit code was preserved and reported.

### Duplicate guard / idempotency fixture — PASS (guard behaviour only)

A test command using exclusive file creation (`O_CREAT | O_EXCL`) was run twice against `agchat-idempotency-marker.txt`.

- First invocation: `EXECUTED_ONCE`
- Second invocation: `DUPLICATE_BLOCKED`
- Read-back: `EXECUTION_COUNT=1`

This demonstrates an application-level idempotency guard in the fixture. It does **not** prove that the MCP extension/proxy itself automatically deduplicates a repeated tool call. That transport-level retry behaviour remains unverified.

### Workspace boundary — PASS for filesystem tools

Initial Desktop Commander config had `allowedDirectories: []`, which means unrestricted filesystem access. This was corrected to:

```json
["/home/dzu/AgChat-Testing/session-2"]
```

The config was read back and confirmed. Listing the test workspace succeeded. Attempting to list `/home/dzu/Documents` was denied with:

```text
Error: Path not allowed: /home/dzu/Documents.
Must be within one of these directories: /home/dzu/AgChat-Testing/session-2
```

This proves the Desktop Commander filesystem-tool boundary. Important limitation: this allow-list is not an OS sandbox and does not confine arbitrary shell commands launched through `start_process`. Do not treat it as protection against malicious shell commands. The Xubuntu host remains a dedicated development machine; avoid destructive commands and sensitive data.


## 11. ChatGPT UI tool-call bridge verification — PASS

After launching Chrome with CDP against a **copy of the user's existing Default profile** (the original profile directory was not used by the CDP test process), the MCP SuperAssistant settings were checked in its own options page:

- Extension version: `0.7.3`.
- MCP server URL: `http://localhost:3006/sse`.
- Transport: `SSE`.
- Extension status: `Connected`.
- Default AI platform was changed from Gemini to `ChatGPT` in the copied profile.

A tool call was then generated by the existing ChatGPT conversation **“^^ Ikuti protokol MCP”** and rendered as a SuperAssistant function card. The card showed:

- Tool: `desktop-commander.read_file`
- Call ID: `9`
- Path: `/home/dzu/AgChat-Testing/session-2/agchat-write-test.txt`
- Offset: `0`
- Length: `10`

The Run control was clicked through the ChatGPT page UI. SuperAssistant displayed a successful result panel:

```text
Result
170 chars · 48ms
[Reading 3 lines from start (total: 3 lines, 0 remaining)]

AgChat TEST A Session 2 - EDIT PASS
MCP file edit and read-back verification.
Git verification fixture change.
```

The result panel had success status (`exec-success`) and an OK result indicator. This is direct evidence that a tool call originating in the ChatGPT conversation was executed by the local Desktop Commander through SuperAssistant and the SSE proxy. The file was only read; no file mutation occurred during this check.

**Result-return mechanism verified:** the extension's `Insert` action queues the result in its internal input-chip mechanism (the ChatGPT adapter calls `tryCapture()` and labels the payload as `tool-result`). The result does not immediately appear in the visible composer. The first `Enter` flushes the queued chip into the composer; the second `Enter` submits it. ChatGPT then receives the `<function_result call_id="9">` payload and continued the same conversation with a substantive `MCP READ PASS` response based on the returned file contents. This proves a complete manually mediated round trip. It does **not** prove fully automatic submission: the Insert + two Enter actions were performed by the test operator through Puppeteer/UI automation.

## 12. Interpretation

This session proves that ChatGPT UI-originated Desktop Commander tool calls can be executed by the installed Xubuntu host through MCP SuperAssistant and the local SSE proxy. It also proves a manually mediated result-return loop: Insert queues a tool-result chip, the first Enter flushes it into the composer, and the second Enter submits it. ChatGPT then continues with a response grounded in the returned result.

A harmless Python process was executed through the bridge using `desktop-commander.start_process`. It emitted distinct stdout and stderr markers and exited intentionally with code 7. `start_process` returned PID 61954 and both markers; the exact exit code was independently verified by calling `desktop-commander.read_process_output` for that PID. The result returned stdout `STDOUT_MARKER`, stderr `STDERR_MARKER`, exit code `7`, runtime `0.13s`, and process state `Completed`. This confirms command execution, output-channel capture, and non-zero exit-code verification through the MCP bridge.

Remaining gaps: repair and verify the Auto Insert/Auto Submit service path; transport retry/deduplication; and further boundary validation. A follow-up read_file call confirmed the file result was produced, but the result panel still exposed the manual Insert button. Although localStorage preferences showed autoInsert=true and autoSubmit=true, the page had no `window.__mcpAutomationState` even after reload. Therefore the automation settings being stored is NOT evidence that the AutomationService is active. A generated assistant response claiming Auto Insert/Auto Submit PASS was contradicted by direct browser UI inspection and is explicitly not accepted as test evidence.

Session 1's direct RDC/Desktop Commander coding-loop results remain separate evidence and must not be conflated with Session 2's browser-extension/proxy bridge results.

## 13. Next actions

Continue in the existing workspace `/home/dzu/AgChat-Testing/session-2/`:

1. Diagnose why `AutomationService` is not exposed (`window.__mcpAutomationState === null`) despite service initialization being present in the extension bundle; inspect extension initialization and adapter readiness before changing code.
2. Verify Auto Insert and Auto Submit through a new actual read-only MCP call; require UI evidence that the result is inserted and submitted without operator action.
3. Test transport retry/deduplication with a harmless idempotency fixture; do not assume the fixture guard proves transport-level deduplication.
4. Assess whether the configured filesystem scope is sufficiently demonstrated.

Do not repeat successful setup or Session 1 tests. Do not label overall TEST A as PASS until the remaining acceptance criteria have evidence.


## 14. Correction and final automation verification — PASS (patched test copy)

**Correction date:** 28 November 2026

The earlier observation that `window.__mcpAutomationState` was null in the page's default JavaScript world was not valid evidence of extension-service failure. The content script runs in an isolated extension execution world. After inspecting the correct isolated context, the test extension was confirmed to have:

- `autoInsert=true`
- `autoSubmit=true`
- `autoExecute=false`
- Active `mcpAdapter` and `mcpClient`
- Submit selector including `button[aria-label="Hantar"]`

### Root cause confirmed

The original extension's `ChatGPTAdapter` submit selector included English labels such as `Send`, but the active ChatGPT UI is localized and exposes the submit button as `button[aria-label="Hantar"]`. The selector did not match that button. A one-selector patch was prepared in a separate unpacked test copy. The original installed extension was not modified.

### Patched test extension

- Display name: `MCP SuperAssistant - AgChat TEST A`
- Version: `0.7.3.1`
- Extension ID: `ccmhmhnpkjkdcdggbcafbhkmeekcgeoo`
- Test copy directory: `/home/dzu/AgChat-Testing/session-2/mcp-superassistant-extension-test/`
- Original extension ID remains: `kngiafgkdnlkgmefdafaibkibegkcaef`
- Existing Chrome profile was retained; no new profile was created.
- The patched selector was verified in the extension isolated execution context.

### End-to-end result-return test — PASS

A new ChatGPT-originated call was generated and the MCP card's **Run** action was activated once to execute the read-only tool. No Insert action or manual Send/Hantar action was performed.

- Tool: `desktop-commander.read_file`
- Call ID: `16`
- Path: `/home/dzu/AgChat-Testing/session-2/agchat-write-test.txt`
- Returned content:
  ```text
  AgChat TEST A Session 2 - EDIT PASS
  MCP file edit and read-back verification.
  Git verification fixture change.
  ```

Observed UI evidence after tool completion:
1. The function result payload was automatically placed in the ChatGPT composer — **Auto Insert PASS**.
2. After the automation delay, the composer became empty and ChatGPT displayed a new assistant response containing the returned file contents — **Auto Submit PASS**.
3. No operator Insert click or manual message submission was used.
4. The operation was read-only; the fixture file was not changed.

### Updated acceptance status

| Check | Result | Evidence |
|---|---|---|
| Auto Insert (patched test copy) | PASS | Function result call ID 16 appeared in the composer without operator Insert action. |
| Auto Submit (patched test copy) | PASS | Composer emptied and ChatGPT produced a subsequent assistant response without operator Send/Hantar action. |
| Original extension compatibility | FAIL until patch is adopted | Original selector lacks the localized `Hantar` selector; original extension was left unchanged. |
| Auto Execute | OFF / NOT TESTED | Preference remained false; no automatic execution of tool cards was enabled. |

**Interpretation:** Auto Insert and Auto Submit are now verified end-to-end on the isolated patched test copy, using the user's existing Chrome profile. This does not mean the original extension has been patched or that Auto Execute is enabled. Keep the original extension untouched until the patch is reviewed and a separate adoption decision is made.

The earlier Section 9 and Section 12 statements that Auto Insert/Auto Submit remained unverified are superseded by this correction for the patched test copy only. The earlier manually mediated round-trip evidence remains valid as a historical test of the original extension.


## 15. MCP SSE duplicate-request replay test — PASS (replay accepted; no transport deduplication)

**Date:** 28 November 2026  
**Target:** Local MCP SuperAssistant proxy at `127.0.0.1:3006`, SSE transport.  
**Safety:** One read-only request followed by a controlled synthetic process invocation. The process used an atomic `O_CREAT | O_EXCL` marker file in the disposable Session 2 workspace.

### 15.1 Same JSON-RPC ID, read-only request

Two identical POST requests were sent through one SSE session, both carrying JSON-RPC request ID `77123` and calling:

- Tool: `desktop-commander.read_file`
- Path: `/home/dzu/AgChat-Testing/session-2/agchat-write-test.txt`

Both POSTs were accepted with HTTP `202 Accepted`. The SSE stream returned **two separate JSON-RPC responses** with ID `77123`, each containing the same three-line file content.

Result: **PASS — duplicate request ID replay was accepted and two responses were emitted.** No file was modified.

### 15.2 Same JSON-RPC ID, controlled side-effect guard

Two identical POST requests were then sent through one SSE session, both carrying JSON-RPC request ID `77125` and invoking `desktop-commander.start_process` with the same Python command. The command attempted to create:

`/home/dzu/AgChat-Testing/session-2/agchat-transport-duplicate-marker.txt`

using atomic `os.open(path, O_CREAT | O_EXCL | O_WRONLY, 0o600)`.

Actual results:

| Item | Observed result |
|---|---|
| First POST | HTTP 202 Accepted |
| Second POST with same JSON-RPC ID | HTTP 202 Accepted |
| First tool result | PID 84188; `EXECUTED_ONCE` |
| Second tool result | PID 84200; `DUPLICATE_BLOCKED` |
| SSE responses | Two separate responses, both ID `77125` |
| Marker read-back | Exactly `EXECUTED_ONCE` |

Result: **PASS — both duplicate submissions reached downstream process execution; the application-level atomic guard prevented the second side effect.**

### 15.3 Interpretation and limits

- The proxy's SSE handler processes each incoming message with a request ID and calls `serverManager.handleRequest(msg)`; the `tools/call` branch forwards the call to the connected MCP server. No request-ID deduplication was observed in this path.
- Replaying an identical JSON-RPC ID does **not** by itself cause the proxy to suppress the second request.
- The atomic marker guard is application-level protection. It does not establish transport-level exactly-once execution.
- This was a deliberate replay test, not a simulated network interruption or an automatically triggered retry. Automatic retry behaviour after connection loss, timeout, or lost response remains **NOT TESTED**.
- For operations with side effects, clients and tools should use application-level idempotency keys/guards and avoid assuming that a repeated JSON-RPC ID is deduplicated by this proxy.

**Acceptance update:** Duplicate-request replay test — PASS (two requests forwarded; duplicate side effect blocked by fixture guard). Automatic transport retry after failure — NOT TESTED. Transport-level deduplication — NOT PRESENT / NOT OBSERVED in the inspected SSE `tools/call` path.


## 16. Downstream stdio server disconnect and reconnect — PASS

**Date:** 28 November 2026  
**Proxy PID:** `47092`  
**Downstream Desktop Commander before test:** PID `47214`

### Procedure

The proxy source was inspected before the controlled test. For stdio servers, the manager attaches an `onclose` handler; when the downstream client closes, it marks the server disconnected and schedules reconnection. The initial reconnect delay is 1 second, with exponential backoff capped at 30 seconds.

A controlled SIGTERM was sent only to the proxy-owned Desktop Commander child process PID `47214`. The proxy process itself (PID `47092`) was left running.

### Actual evidence

- Before: proxy-owned Desktop Commander PID `47214`.
- Action: `kill -TERM 47214`.
- Approximately four seconds later: a replacement Desktop Commander process appeared as PID `84533`.
- Proxy PID `47092` remained alive.
- A fresh SSE `tools/list` request then returned **25 tools**.
- Both `desktop-commander.read_file` and `desktop-commander.start_process` were present in the returned tool list.

Result: **PASS — the proxy detected the downstream stdio server closure, restarted/reconnected its configured server, and restored tool discovery without restarting the proxy process.**

### Scope and limits

This verifies recovery from a downstream stdio child-process exit. It does not verify automatic replay of an in-flight tool call, recovery after browser-to-proxy SSE disconnection, or exactly-once semantics. Those remain separate behaviours and must not be inferred from this reconnect test.


## 17. Chrome extension ↔ proxy SSE reconnect after full proxy restart — PASS (session re-established)

**Date:** 28 November 2026  
**Test extension:** `MCP SuperAssistant - AgChat TEST A 0.7.3.1`  
**Proxy endpoint:** `http://127.0.0.1:3006/sse`

### Procedure

1. Confirmed the proxy was listening on port 3006 and its downstream Desktop Commander server was connected.
2. Sent SIGTERM to the proxy process PID `47092`, leaving Chrome and its existing profile open.
3. Verified port 3006 became closed.
4. Restarted the proxy from the same Session 2 directory using the same `config.json`, without restarting Chrome or changing the extension.
5. Inspected the new proxy log at `/home/dzu/AgChat-Testing/session-2/proxy-restart-test.log`.

### Observed evidence

- The new proxy process initialized Desktop Commander and reported 25 tools and 2 resources.
- The new proxy log recorded new SSE connections from `127.0.0.1`.
- A new SSE session was created: `4c3f579e-5775-4a99-a94e-dbb0919f8ec6`.
- On that session, the extension/client sent `initialize` (protocol version `2025-11-25`), followed by `notifications/initialized`.
- The same new session then issued repeated `resources/list`, `tools/list`, and `ping` requests. This shows that the client established a new SSE session after the proxy process restart and resumed MCP discovery/keepalive traffic.
- Chrome was not restarted and the test extension remained installed/enabled.

Result: **PASS — Chrome extension/client re-established an SSE session with the restarted proxy and resumed MCP initialization, discovery, and keepalive.**

### Scope and limitations

- This verifies reconnection after a full proxy process restart, not merely a downstream Desktop Commander child restart.
- It verifies new session establishment and repeated MCP discovery/keepalive traffic. It does **not** prove that an in-flight request during the outage is replayed, nor exactly-once execution.
- A post-reconnect tool invocation from the ChatGPT UI was not conclusively observed in this run; therefore end-to-end tool-call recovery after the restart remains **NOT VERIFIED**.
- Automatic SSE reconnect is supported by the EventSource implementation used by the extension's SSE client. The observed new session and subsequent MCP messages are the runtime evidence for this test.

**Acceptance update:** Browser/client ↔ proxy SSE session re-establishment — PASS. In-flight request replay — NOT TESTED. Post-reconnect UI tool invocation — NOT VERIFIED.


## 18. Correction — post-restart ChatGPT UI tool call verified PASS

**Date:** 28 November 2026

Section 17 initially stated that the post-reconnect UI tool invocation was not verified. A subsequent inspection of the active ChatGPT conversation confirmed that the tool card had completed successfully and ChatGPT had received the actual file contents.

### Evidence

- ChatGPT conversation: `6b0a6dd3-8a04-83ec-b94b-71e72d826121`
- Tool: `desktop-commander.read_file`
- Target: `/home/dzu/AgChat-Testing/session-2/agchat-write-test.txt`
- ChatGPT UI displayed the MCP tool card as completed, with a result payload (170 chars, 39 ms).
- ChatGPT's subsequent assistant response reproduced the three lines returned from Xubuntu and reported the MCP invocation, Desktop Commander response, file access, and result-return as PASS.
- Composer was empty after the assistant response.
- File remained read-only and unchanged.
- Independent direct SSE verification after restart also returned JSON-RPC ID `99201` with the same three-line file content.

### Corrected acceptance status

| Check | Corrected result |
|---|---|
| Proxy process restart | PASS |
| New extension/client SSE session | PASS |
| MCP initialize + discovery + keepalive | PASS |
| Post-restart ChatGPT UI tool invocation | PASS |
| Desktop Commander read_file result-return | PASS |
| File integrity (read-only) | PASS |
| In-flight request replay across restart | NOT TESTED |
| Exactly-once execution across restart | NOT TESTED |

**Correction:** The earlier "post-reconnect UI tool invocation — NOT VERIFIED" statement in Section 17 is superseded by this Section 18. The end-to-end read-only tool path is now verified after a full proxy restart. This does not establish replay or exactly-once semantics for an in-flight side-effecting operation.


## 19. Post-restart `desktop-commander.start_process` via MCP SSE — PASS

**Date:** 28 November 2026  
**Proxy:** restarted instance listening on `127.0.0.1:3006`.

### Procedure and evidence

After the full proxy restart, a fresh SSE session was opened and a JSON-RPC `tools/call` request (ID `99202`) invoked `desktop-commander.start_process` with a harmless Python command:

`python3 -c 'print("POST_RESTART_PROCESS_PASS"); print(2+3)'`

Observed response:

- HTTP POST: `202 Accepted`
- JSON-RPC response ID: `99202`
- Tool result: process started (PID `87074`)
- Captured stdout: `POST_RESTART_PROCESS_PASS` and `5`
- Process completed successfully.

Result: **PASS — post-restart proxy → Desktop Commander → start_process → captured output/result path is working.**

### Scope clarification

This is a direct MCP SSE invocation after restart. It verifies the tool and result path through the proxy, but it is not evidence of a ChatGPT UI-originated `start_process` invocation. ChatGPT UI read_file after restart was separately verified in Section 18. A UI-originated start_process call remains unverified.


## 20. Post-restart `write_file` + independent read-back — PASS

**Date:** 28 November 2026  
**Proxy:** restarted instance, `127.0.0.1:3006)  
**Fixture:** `/home/dzu/AgChat-Testing/session-2/agchat-write-after-restart.txt`

### Procedure

1. Invoked `desktop-commander.write_file` through a fresh MCP SSE session (JSON-RPC ID `99203`).
2. Wrote a dedicated three-line fixture containing `WRITE_OPERATION_PASS`.
3. Verified the file independently on the host using `cat`, `sha256sum`, and Git status.
4. Opened a separate fresh SSE session and invoked `desktop-commander.read_file` (JSON-RPC ID `99206`) to read the newly written file back through MCP.

### Evidence

- `write_file`: HTTP `202 Accepted`; result text: `Successfully wrote ... (3 lines)`.
- Host read-back: exact three lines present.
- SHA-256: `3be7b3f8edd060c36aed8ee8ad31c786ad5a7bc35bcce791d816748c2fec7c78`.
- `git diff --check`: exit 0 (no whitespace errors).
- `read_file` in a separate SSE session: HTTP `202 Accepted`; JSON-RPC result returned the exact three-line fixture including `WRITE_OPERATION_PASS`.
- Git status showed the fixture as untracked (`?? agchat-write-after-restart.txt`), as expected for a newly created file. Existing unrelated Session 2 artifacts remained untouched by this test.

Result: **PASS — post-restart write operation, host persistence, independent MCP read-back, and basic Git whitespace validation all succeeded.**

### Scope

This is a controlled local fixture write in the disposable Session 2 workspace. It does not verify writes from ChatGPT UI after restart, atomicity for concurrent writes, or rollback/recovery after a partial write.


## 21. ChatGPT UI-originated `write_file` after proxy restart — EXECUTION PASS / CONTENT PARTIAL

**Date:** 28 November 2026  
**Route:** ChatGPT UI → MCP SuperAssistant extension → proxy `127.0.0.1:3006` → Desktop Commander  
**Target:** `/home/dzu/AgChat-Testing/session-2/agchat-chatgpt-ui-write.txt`

### Procedure and evidence

1. Submitted a request in the existing ChatGPT conversation asking for `desktop-commander.write_file` to create a new fixture in the Session 2 workspace.
2. ChatGPT generated a `desktop-commander.write_file` tool card. The tool was explicitly run from the ChatGPT UI using its **Run** control.
3. The extension returned a completed result card (95 chars; 105 ms).
4. ChatGPT displayed the actual Desktop Commander result: `Successfully wrote to .../agchat-chatgpt-ui-write.txt (1 lines)`.
5. Verified on the Xubuntu host: file exists, contains exactly the single line `AGCHAT CHATGPT UI WRITE PASS`, 28 bytes, zero newline characters. SHA-256: `82d246dedf7c798ef06ea8ec3f71397893029bade8d7d325e9f696e77969093c`.
6. Git status reports the new fixture as untracked; `git diff --check` returned no errors.

### Result

- **PASS:** UI-originated MCP tool invocation, extension Run flow, proxy routing, Desktop Commander write operation, and host persistence.
- **PARTIAL / CONTENT ASSERTION NOT MET:** The original test expected three lines, but ChatGPT generated a one-line `content` argument. The tool's success result therefore does not establish correct multi-line payload handling from the UI.
- No other file was requested for modification. The file is a disposable fixture.

### Scope and follow-up

This validates the ChatGPT UI → extension → proxy → Desktop Commander write path after proxy restart for a one-line payload. Multi-line payload fidelity from the UI remains unverified. The earlier direct-SSE multi-line write/read-back test in Section 20 remains a separate PASS and must not be conflated with this UI-originated result.


## 22. ChatGPT UI-originated `read_file` after proxy restart — NOT VERIFIED

**Date:** 28 November 2026  
**Target:** `/home/dzu/AgChat-Testing/session-2/agchat-chatgpt-ui-write.txt`

### Attempt

Three prompts were submitted in the existing ChatGPT UI conversation requesting `desktop-commander.read_file` for the target fixture. The assistant completed its turns, but no new `desktop-commander.read_file` tool card was generated. Therefore there was no UI **Run** action and no extension result-return event for this read request.

### Independent host observation (not an MCP UI test)

- Host `cat -A` confirmed the file contains `AGCHAT CHATGPT UI WRITE PASS`.
- File size: 28 bytes; line count: 0 (the file has no trailing newline).
- SHA-256 remains `82d246dedf7c798ef06ea8ec3f71397893029bade8d7d325e9f696e77969093c`.
- Proxy listener remained active on `127.0.0.1:3006`.
- Fixture remains untracked; no other file was changed by this verification.

### Result and interpretation

**NOT VERIFIED / BLOCKED AT TOOL-CALL GENERATION.** This is not evidence of a proxy or Desktop Commander read failure: the UI never produced a read tool call to execute. The host observation confirms file persistence only and must not be represented as an MCP read-back PASS.

Next diagnostic should focus on why ChatGPT is not generating a `read_file` card in this conversation, despite the earlier successful UI-originated `write_file` card. Do not alter the proxy or extension until tool-call generation is isolated.


## 23. AutomationService / ChatGPTAdapter source audit — BUG CONFIRMED, runtime fix not yet applied

**Date:** 28 November 2026  
**Scope:** Read-only audit of the isolated unpacked TEST A extension copy at `/home/dzu/AgChat-Testing/session-2/mcp-superassistant-extension-test/`. No extension source, proxy configuration, Chrome profile, or runtime preference was modified during this audit.

### 23.1 ChatGPT adapter selector

The TEST A copy's `ChatGPTAdapter` selector list already contains:

```js
button[aria-label="Hantar"]
```

alongside the English send selectors. This confirms the localized selector patch is present in the audited test copy. It does not by itself prove that the full Auto Submit automation path is running.

### 23.2 Auto Insert and Auto Submit orchestration

The bundled `AutomationService.handleToolExecutionComplete()` reads the preference values and follows this sequence:

1. If `autoInsert` is enabled, call `handleAutoInsert()`.
2. Only if insertion succeeds, and `autoSubmit` is also enabled, schedule submission using `scheduleCoalescedSubmit()`.
3. The coalesced scheduler waits while MCP execution/result panels still indicate pending work, then calls `handleAutoSubmit()`.
4. `handleAutoSubmit()` obtains the active adapter's `submitForm` method and invokes it after the configured delay and an additional 800 ms wait.

The ChatGPT adapter's `submitForm()` searches its selector list, checks that the button exists, is enabled, and is visible, then calls `click()`. The service therefore has an explicit Auto Submit path; merely inserting a result into the composer is not the full expected behaviour.

### 23.3 Confirmed preference-toggle defect

The TEST A content bundle contains the following defect in `createToggleStateManager()`:

- `setAutoInsert(value)` writes `updatePreferences({autoSubmit: value})` instead of updating `autoInsert`.
- `getState()` reads `preferences.autoSubmit` into a shared value and returns that value for both `autoInsert` and `autoSubmit`.

Consequences:
- The popover's Auto Insert and Auto Submit controls are not independently mapped to their respective preference keys.
- Toggling Auto Insert through this manager can overwrite the Auto Submit preference.
- The popover can report the same state for both controls even when the stored preferences are intended to differ.

The UI store's generic `updatePreferences()` method correctly merges the supplied keys; the defect is in the toggle manager passing the wrong key and reading the same key twice.

### 23.4 Important distinction

The AutomationService itself reads `preferences.autoInsert` and `preferences.autoSubmit` separately. Therefore the toggle-manager defect is confirmed, but it is not sufficient by itself to prove that every route for changing preferences is broken. A separate UI/settings path may write the two keys independently. Runtime preference values and the exact control used to set them must be inspected before concluding why a particular run did not submit.

### 23.5 Audit outcome and next test

| Item | Result |
|---|---|
| ChatGPT adapter includes localized `Hantar` selector in TEST A copy | PASS (source inspection) |
| ChatGPT adapter has explicit `submitForm()` implementation | PASS (source inspection) |
| AutomationService sequences successful Auto Insert before Auto Submit | PASS (source inspection) |
| Scheduler waits for pending tool/result execution before submission | PASS (source inspection) |
| Popover `setAutoInsert` writes the wrong preference key | BUG CONFIRMED |
| Popover `getState` conflates Auto Insert and Auto Submit | BUG CONFIRMED |
| Runtime Auto Insert/Auto Submit state after toggling controls | NOT TESTED |
| Actual Auto Insert → Auto Submit on current test copy after this audit | NOT TESTED |

**Next action:** inspect the active extension's isolated-world preference state and test the two controls independently, without changing source code. If the defect is reproduced, prepare a minimal patch in the isolated TEST A copy only, then verify separate preference values and a read-only end-to-end Auto Insert → Auto Submit run. Do not modify the original installed extension or proxy as part of this test.
