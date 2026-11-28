# TEST 43 — SuperAssistant Proxy + Orchestrator E2E Result

**Result: PASS (end-to-end)**  
**Date:** 28 November 2026 (MYT)  
**Host:** Xubuntu `dzu-miniPC`  
**Workspace:** `/home/dzu/AgChat-Testing/session-2/`  
**Chat:** The originating ChatGPT conversation supplied for this test  
**Approval mode:** Auto Execute OFF; each tool card was run manually.

## 1. Objective

Verify the complete route:

`ChatGPT conversation → Chrome MCP SuperAssistant extension → local SSE proxy → AgChat Orchestrator MCP server → Desktop Commander MCP → Xubuntu workspace`

The test must prove a tool call initiated in the originating ChatGPT conversation reaches the Orchestrator through the SuperAssistant proxy, invokes Desktop Commander, and returns a verifiable result to that same conversation.

## 2. Environment and configuration

- Browser: Google Chrome Stable `158.0.8127.88`.
- Browser profile: dedicated clone `chrome-user-profile-clone` based on the Chrome Default profile.
- CDP endpoint: `http://127.0.0.1:9222`.
- Test extension ID: `ccmhmhnpkjkdcdggbcafbhkmeekcgeoo` (isolated TEST A unpacked copy).
- Proxy package: `@srbhptl39/mcp-superassistant-proxy`, launched with `--outputTransport sse`.
- Proxy endpoint: `http://localhost:3006/sse`, listener `127.0.0.1:3006`.
- Configuration: `config-test43.json`, containing two MCP servers:
  - `desktop-commander` — 25 tools.
  - `agchat-orchestrator` — 5 tools.
- Orchestrator: `agchat-orchestrator-mcp.mjs`; JavaScript syntax check passed with `node --check`.

## 3. Operation and approval sequence

Operation key:

`agchat-test43-proxy-e2e-20261128-01`

The Orchestrator's SQLite ledger recorded:

1. `SUBMITTED` — intent persisted as `PENDING_APPROVAL`, with `auto_execute=false`.
2. `APPROVED` — explicit approval tool call changed state to `PENDING`.
3. `EXECUTION_STARTED` — Orchestrator invoked Desktop Commander `start_process` for the fixed harmless fixture.
4. Initial status was recorded as `FAILED` because the Orchestrator expected a JSON proof marker but Desktop Commander returned a process wrapper containing the JSON result.
5. The fixture had nevertheless created the proof file. Its output reported `CREATED`, 29 bytes, with the expected SHA-256.
6. The Orchestrator was patched to support structured process output and a reconciliation operation was added.
7. ChatGPT then invoked `agchat-orchestrator.agchat_reconcile_test43` through SuperAssistant. The tool used Desktop Commander `read_file` to inspect the existing proof file. It did **not** rerun the fixture.
8. The ledger recorded `RECONCILED_SUCCEEDED`; final operation state is `SUCCEEDED`, `approved=1`.

## 4. End-to-end evidence

The proxy log contains the incoming MCP call:

`tools/call → agchat-orchestrator.agchat_reconcile_test43`

The Orchestrator result confirms:

- `executor: desktop-commander.read_file`
- `reconciled_existing_effect: true`
- `proof_path: /home/dzu/AgChat-Testing/session-2/agchat-test43-proxy-e2e-proof.txt`
- `proof_bytes: 29`
- `state: SUCCEEDED`

Proof file content:

```text
AGCHAT_TEST43_PROXY_E2E_PASS
```

SHA-256:

`172b5e80f247d214a99166664f4738892ea2894ebc0e8475a2865e3e82c951eb`

Local evidence files:

- `proxy-test43.log` — proxy startup, two MCP server handshakes, tool counts and the incoming Orchestrator call.
- `agchat-test43-orchestrator.sqlite` — operation state and event history.
- `agchat-test43-proxy-e2e-proof.txt` — proof marker.
- `screenshots/test43-chrome-chat.png` — Chrome profile and originating ChatGPT test conversation.

## 5. Safety and execution control

- Auto Execute remained OFF.
- The tool's visible `Run` button was activated manually after confirming exactly one visible Run control.
- No second fixture execution was performed during reconciliation.
- The reconciliation operation only read the existing proof file through Desktop Commander and corrected the ledger's false-negative status.
- The original proof file was preserved; no overwrite or deletion occurred.

## 6. Conclusion

**TEST 43 PASS.** The complete ChatGPT → SuperAssistant → SSE proxy → Orchestrator → Desktop Commander route is demonstrated. The initial false-negative exposed a response-parsing defect in the Orchestrator, not a failure of the Desktop Commander side effect. The existing proof was independently read through the same MCP route and the operation ledger was reconciled to `SUCCEEDED`.

This pass establishes the tested route and this fixed fixture. It does not establish transport-level exactly-once semantics or general-purpose duplicate suppression for arbitrary tools.
