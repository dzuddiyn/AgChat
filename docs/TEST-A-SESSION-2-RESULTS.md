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

Remaining gaps: fully automatic result submission without operator action, transport retry/deduplication, and further boundary validation. Do not conflate direct RDC checks with bridge tests.

Session 1's direct RDC/Desktop Commander coding-loop results remain separate evidence and must not be conflated with Session 2's browser-extension/proxy bridge results.

## 13. Next actions

Continue in the existing workspace `/home/dzu/AgChat-Testing/session-2/`:

1. Investigate whether the Insert → chip flush → submit sequence can be safely automated without modifying the extension's installed code.
2. Test transport retry/deduplication with a harmless idempotency fixture; do not assume the fixture guard proves transport-level deduplication.
3. Assess whether the configured filesystem scope is sufficiently demonstrated.

Do not repeat successful setup or Session 1 tests. Do not label overall TEST A as PASS until the remaining acceptance criteria have evidence.
