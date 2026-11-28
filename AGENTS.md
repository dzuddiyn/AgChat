# AGENTS.md — AgChat operating instructions

These instructions apply to agents and assistants working in this repository and the associated Xubuntu SuperAssistant test environment.

## 1. Browser policy — mandatory

- Use **Google Chrome only**, with the **Rdeeen** Chrome profile.
- The dedicated working profile is:
  `/home/dzu/AgChat-Testing/session-2/chrome-user-profile-clone`
- This profile is a working clone of the Chrome profile signed in as `Rdeeen@gmail.com`; it preserves the test extension and browser session.
- Do **not** launch, switch to, or use Vivaldi for AgChat work.
- Do not use Firefox or another browser as a substitute.
- Before browser automation, verify the active Chrome process has the expected `--user-data-dir`, then inspect CDP targets and select the exact ChatGPT conversation.
- CDP endpoint in the current setup: `http://127.0.0.1:9222`.
- Do not start another Chrome instance with a conflicting debugging port or a different profile. Reuse the already-running Rdeeen profile session.
- Do not change the user's original Chrome profile or install/modify the original Web Store extension when an isolated test copy is intended.

## 2. Development method

The task is to build, configure, test and troubleshoot **MCP SuperAssistant on Xubuntu**. RDC, CDP, GUI automation, shell/process inspection and Git are our development instruments; they are not the SuperAssistant product itself.

- RDC is the host-control plane for files, processes, ports, logs and screenshots.
- CDP controls Chrome tabs, DOM, extension service-worker contexts and browser state.
- `agchat-gui.py` and X11 utilities handle native desktop/window operations.
- Use AT-SPI2 where the native accessibility tree is exposed.
- Use the SuperAssistant extension and local MCP proxy to validate the actual ChatGPT-to-MCP route.

Detailed procedure: [docs/AGCHAT-WORKING-METHOD.md](docs/AGCHAT-WORKING-METHOD.md).

## 3. Continue from verified checkpoints

- Do not repeat setup or tests already marked PASS unless new evidence invalidates them.
- Current verified milestone: **Test 43 PASS** — ChatGPT → SuperAssistant → SSE proxy → AgChat Orchestrator → Desktop Commander.
- **Test 44 PARTIAL** — controlled failure (exit 7) and success (exit 0) verified through the same MCP route; duplicate guard output and state count 1 verified, duplicate exit 17 not recoverable after process session expiry.
- **Test 45 PASS** — out-of-scope filesystem listing `/home/dzu/Documents` denied through the same proxy route; this does not sandbox shell execution.
- **Test 46 PASS** — two Python files created through MCP, four unit tests passed (exit 0), both files read back and targeted Git status/hashes recorded.
- **TEST A V1 PoC gate: PASS WITH LIMITATIONS.** Generic transport exactly-once, proxy reconnect recovery and OS-level shell isolation are not claimed.
- Test 43 evidence: [docs/TEST-43-PROXY-ORCHESTRATOR-RESULTS.md](docs/TEST-43-PROXY-ORCHESTRATOR-RESULTS.md).
- Test 44 evidence: [docs/TEST-44-COMMAND-FAILURE-DUPLICATE-RESULTS.md](docs/TEST-44-COMMAND-FAILURE-DUPLICATE-RESULTS.md).
- Overall TEST A remains PARTIAL. Continue with the first outstanding acceptance item in [docs/TEST-A-PLAN.md](docs/TEST-A-PLAN.md), not with browser installation or Test 43 replay.
- Inspect current process, port, ledger and file state before any side-effecting action. Never rerun a fixture because a response is delayed or ambiguous; inspect its original PID, output, ledger and proof first.

## 4. Approval and safety

- **Auto Execute must remain OFF.**
- Tool execution requiring approval must be manually reviewed and run; never enable automatic execution to speed up testing.
- Preserve the user's files and original browser profile. Use the disposable AgChat workspace.
- Do not repeat writes/process launches where the prior effect may already have occurred.
- `allowedDirectories` constrains Desktop Commander filesystem tools; it is not an OS sandbox for arbitrary shell commands.
- Do not expose credentials, cookies, tokens or profile secrets in logs or documentation.

## 5. Communication and evidence

- Keep visible replies concise; do not reduce the actual investigation, execution, verification or documentation.
- Avoid narrating every tool call and avoid repeating completed setup.
- Distinguish PASS, PARTIAL, FAIL and NOT TESTED. A proxy startup or tool discovery alone is not end-to-end proof.
- Record exact commands/configuration where useful, file paths, tool route, exit/result status, hashes and Git commit references.
- Update this repository's SoT when verified evidence changes the status or working method.
