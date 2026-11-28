# TEST 44 — Controlled command execution, failure and duplicate guard

**Result: PARTIAL (28 November 2026).**  
**Route:** originating ChatGPT conversation → SuperAssistant extension → local SSE proxy → Desktop Commander MCP → Xubuntu test workspace.  
**Browser:** Google Chrome Rdeeen profile clone only.  
**Approval:** Auto Execute OFF; each tool card was manually run.

## 1. Scope

Continue after Test 43 without repeating its setup. Verify controlled process execution, honest expected-failure reporting, successful file creation and a duplicate attempt that does not produce a second side effect.

## 2. Fixture

Created a new isolated fixture:

- Script: `/home/dzu/AgChat-Testing/session-2/agchat-test44-fixture.py`
- State: `/home/dzu/AgChat-Testing/session-2/agchat-test44-state.json`
- Modes: `expected-failure`, `success`, `duplicate`
- State creation uses `os.O_CREAT | os.O_EXCL` to reject a second write.

The Test 43 fixture and proof file were not modified.

## 3. Results

### 3.1 Expected failure — PASS

Command executed through SuperAssistant MCP:

```bash
python3 /home/dzu/AgChat-Testing/session-2/agchat-test44-fixture.py expected-failure
```

Evidence returned through the ChatGPT tool result:

- stdout: `TEST44_STDOUT_EXPECTED_FAILURE`
- stderr: `TEST44_STDERR_EXPECTED_FAILURE`
- exit code: `7`
- PID: `124803`

The exit code was obtained by a separate `desktop-commander.read_process_output` call through the same SuperAssistant proxy. The fixture was not rerun.

### 3.2 Success — PASS

Command executed through SuperAssistant MCP:

```bash
python3 /home/dzu/AgChat-Testing/session-2/agchat-test44-fixture.py success
```

Evidence:

- stdout JSON status: `CREATED`
- state file: `agchat-test44-state.json`
- bytes: `67`
- SHA-256: `77c3516e5d163f7a83f7c01633bd159eb9ee0d6fb1502e3e3249751901db1f85`
- stderr: `TEST44_STDERR_SUCCESS_PATH_CLEAR`
- exit code: `0`
- PID: `125823`

The exit code was obtained through a separate `desktop-commander.read_process_output` call via the same proxy. The success fixture was not rerun.

### 3.3 Duplicate attempt — PARTIAL

Command executed once through SuperAssistant MCP:

```bash
python3 /home/dzu/AgChat-Testing/session-2/agchat-test44-fixture.py duplicate
```

Evidence:

- stdout: `DUPLICATE_BLOCKED`
- stderr: `state file already exists; no second side effect`
- PID: `126708`
- subsequent read-back of state file:
  - `execution_count = 1`
  - marker: `AGCHAT_TEST44_SINGLE_EXECUTION`
  - bytes: `67`
  - SHA-256 unchanged: `77c3516e5d163f7a83f7c01633bd159eb9ee0d6fb1502e3e3249751901db1f85`

The follow-up `read_process_output` request was routed through the proxy, but the final exit code `17` was not independently captured in the ChatGPT result. Therefore the duplicate guard's visible result and no-second-side-effect condition are confirmed, but the full duplicate process exit-status criterion remains PARTIAL. The duplicate fixture was not rerun.

## 4. Interpretation

Test 44 demonstrates that the actual ChatGPT → SuperAssistant → proxy → Desktop Commander path can:
- execute a controlled process;
- preserve stdout and stderr markers;
- report expected non-zero exit status (7);
- report a successful exit status (0);
- reject a repeated side-effect attempt at the fixture level;
- preserve the state count at exactly one.

It does **not** prove generic proxy-level exactly-once delivery or universal deduplication for arbitrary commands. The duplicate exit code 17 remains unverified.

## 5. Safety

- Auto Execute remained OFF.
- Each tool card was run manually.
- No fixture was repeated to recover an ambiguous response.
- The duplicate attempt did not alter the state file.
- Test 43 proof and SQLite ledger were left untouched.

## 6. Final disposition

The duplicate exit code 17 could not be recovered: the original Desktop Commander session expired and a later read returned `No session found for PID 126708`. The duplicate fixture was not rerun. This is an exit-status telemetry gap, not evidence of a second side effect: `DUPLICATE_BLOCKED`, `execution_count=1` and unchanged SHA-256 were independently observed.

**Disposition:** fixture-level duplicate guard PASS for the agreed V1 PoC; exit-code telemetry PARTIAL. Generic transport-level exactly-once delivery remains unproven and is not claimed. Test 44 remains labelled PARTIAL to preserve this evidence limitation; TEST A V1 acceptance is PASS WITH LIMITATIONS after Tests 43–46.
