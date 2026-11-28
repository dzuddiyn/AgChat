# AgChat Development Method — Building SuperAssistant on Xubuntu

**Status:** Operational method documented from observed work on 28 November 2026.  
**Scope:** The practical method we use to develop, install, configure, test, troubleshoot and maintain MCP SuperAssistant on the Xubuntu PC. It documents how ChatGPT and the operator use RDC, CDP, desktop GUI automation, shell/process inspection and source/configuration tools to build and validate the setup. It is not a manual describing SuperAssistant's internal runtime behaviour or a general guide to using its tools.  
**Source of Truth:** This document records the working method and verified boundaries. Test-specific results remain in `docs/TEST-A-SESSION-2-RESULTS.md`.

## 1. Operating principle

**Keep user-visible replies concise; do not reduce the work performed.**

ChatGPT should continue to inspect, plan, execute, troubleshoot, verify, and document thoroughly. The visible response should report only the useful outcome, current blocker, evidence, and next action. Avoid narrating every tool call or repeating setup already proven successful.

The originating ChatGPT conversation is the control and reasoning surface. Xubuntu tools are its hands and eyes. Preserve same-chat continuity; do not move work into parallel chats to bypass provider limits, entitlements, or safety policies.

### What “working method” means in this project

The subject is **our development workflow for getting SuperAssistant working on Xubuntu**, not “how SuperAssistant works” as a product. The workflow covers:

1. Establish and verify the Xubuntu development environment.
2. Inspect the installed browser, extension version/ID, proxy package and MCP server configuration before changing anything.
3. Use RDC to inspect host files, dependencies, processes, ports, logs and configuration; create backups before edits.
4. Use CDP to inspect the actual Chrome target, extension service worker/context, page DOM and browser state. Use GUI automation for native desktop/window interactions that CDP cannot control.
5. Make the smallest configuration or code change needed, preserving the original Web Store extension when testing an isolated unpacked copy.
6. Restart only the affected process/service, reconnect the extension, and verify the live connection and discovered tools.
7. Run a narrow test through the intended path, inspect both sides of each boundary, and preserve logs, hashes, screenshots and Git commits.
8. Record what is proven, what failed, and the next checkpoint in the SoT.

**Separation of concerns:** RDC/CDP/GUI are the methods and instruments we use to build and troubleshoot the Xubuntu installation. SuperAssistant extension, proxy and MCP servers are the system components being built/configured/tested. A successful RDC or CDP operation alone does not prove SuperAssistant is working; proof requires the relevant extension/proxy/MCP path to be exercised.

**Communication rule:** visible replies stay short to conserve chat space; actions, verification and SoT updates remain complete. Do not narrate every tool call or repeat already-passed setup.

## 2. Working architecture

The operational workflow uses several complementary control planes:

| Layer | Tool / mechanism | Responsibility |
|---|---|---|
| Reasoning and task control | ChatGPT Chat Mode, same conversation | Understand goal, plan actions, interpret evidence, decide next step |
| Remote host access | Remote Desktop Commander (RDC) | Reach the authorised Xubuntu device; execute commands; inspect files and process output |
| Browser automation | Chrome DevTools Protocol (CDP) | Inspect and control Chrome tabs, DOM, runtime contexts, extension UI and browser state |
| Desktop GUI control | `agchat-gui.py` + X11 utilities | Screenshots, window discovery/focus, pointer, mouse, keyboard |
| Desktop accessibility | AT-SPI2 / PyAT-SPI | Inspect native desktop application/accessibility tree where exposed |
| Browser UI accessibility | DOM / CDP / Puppeteer | Inspect web content and extension popovers; preferred over AT-SPI for Chromium renderer content |
| MCP tool bridge | MCP SuperAssistant extension + local proxy | Present local MCP tools to the originating ChatGPT conversation |
| Local execution | Desktop Commander MCP | File operations, directory listing, process start/output and related host actions |
| Evidence and history | Git, test reports, hashes, SQLite fixtures | Verify changes and preserve reproducible test evidence |

Typical data/control path:

`ChatGPT → SuperAssistant extension → local MCP proxy (127.0.0.1:3006) → MCP server(s) → Desktop Commander → Xubuntu`

For direct remote administration, ChatGPT can use RDC independently. A direct RDC or direct Desktop Commander call is **not** evidence that the browser-extension/proxy route works end-to-end.

## 3. Remote Desktop Commander (RDC) — host control

Use the already-authorised RDC device connection to perform host-side work. The known device identifier in the current test environment is held by the RDC connection; do not hard-code credentials or expose connection secrets in project documentation.

Use RDC for:
- Running read-only inventory and diagnostic commands.
- Starting or stopping known test processes.
- Reading/writing test files where appropriate.
- Reading process output and exit status.
- Running the GUI helper.
- Capturing evidence and checking hashes.
- Inspecting service/listener state and logs.

Operational sequence:
1. Confirm the target device and intended working directory.
2. Prefer read-only inspection first.
3. Use absolute paths for scripts and important files.
4. Start a process with an appropriate timeout; if it continues asynchronously, retain its PID.
5. Retrieve output from that same PID rather than rerunning the command.
6. Verify expected files, output markers, exit code, Git status/diff, and/or SHA-256.
7. Report only what the evidence establishes.

Do not rerun a side-effecting command merely because the first response is delayed or truncated. First inspect the original PID, output, ledger, marker, or file state.

Known working workspace:
`/home/dzu/AgChat-Testing/session-2/`

Known RDC operations used in this workflow include:
- `start_process`
- `read_process_output`
- `write_file`
- `read_file`
- `list_directory`
- `kill_process`

Exact tool schemas must be obtained from the connected tool rather than guessed.

## 4. Chrome DevTools Protocol (CDP) — browser control

CDP is the preferred browser-control plane when the browser is already running with remote debugging enabled. In the tested setup, Chrome exposed its debugging endpoint at:

`http://127.0.0.1:9222`

### Chrome profile and CDP launch discipline

The working browser for SuperAssistant development is the **Google Chrome profile clone** at:

`/home/dzu/AgChat-Testing/session-2/chrome-user-profile-clone`

This clone preserves the test profile's extensions and signed-in browser state while allowing a dedicated CDP endpoint. Launch it with:

```bash
/usr/bin/google-chrome \
  --remote-debugging-port=9222 \
  --user-data-dir=/home/dzu/AgChat-Testing/session-2/chrome-user-profile-clone \
  --profile-directory=Default \
  --no-first-run --no-default-browser-check
```

Use `http://127.0.0.1:9222/json/version` and `/json/list` to verify CDP and select the exact ChatGPT conversation. The normal Chrome data directory may reject remote debugging; use the dedicated clone rather than altering the user's default profile. Do not switch to Vivaldi for this workflow when the requested Chrome profile is available. Confirm the active extension ID before inspecting or patching an extension.

Use CDP to:
- Discover current tabs through `/json/list`.
- Read browser version/debugger metadata through `/json/version`.
- Attach to the specific target's `webSocketDebuggerUrl`.
- Inspect DOM and page state using `Runtime.evaluate`.
- Inspect the DOM tree, including shadow roots, using `DOM.getDocument` with piercing enabled.
- Inspect element bounds and state before clicking.
- Use `Input.insertText` and `Input.dispatchKeyEvent` for controlled keyboard interaction.
- Use `Target.createTarget` and `Target.activateTarget` only when a new/known tab is intentionally needed.
- Inspect extension service workers, extension pages, isolated execution contexts, and popover state when relevant.

Recommended safe CDP pattern:
1. Enumerate targets; identify the exact browser tab by URL/title.
2. Attach to that target, not an arbitrary first tab.
3. Read state before changing it.
4. Prefer DOM-backed semantic actions when possible.
5. For coordinate-based actions, obtain current bounding rectangles and confirm the target is visible and unobstructed.
6. After a mutation, read the state back and capture a screenshot if visual confirmation matters.
7. Close the CDP connection cleanly.

Important distinction:
- CDP controls browser tabs and Chromium runtime.
- It does not replace full desktop control: window focus, native dialogs, other applications, and the whole Xfce desktop may require the GUI helper.
- CDP DOM/runtime inspection is not proof that a user-visible click was successfully delivered. Verify the resulting UI state.

### Browser/extension troubleshooting lessons

- The original installed Web Store extension and the isolated TEST A unpacked extension are different installations. Inspect the active extension ID and target context before patching or toggling anything.
- The original Web Store extension ID observed in the test was `kngiafgkdnlkgmefdafaibkibegkcaef`; the isolated TEST A extension ID observed in the test was `ccmhmhnpkjkdcdggbcafbhkmeekcgeoo`. Do not confuse them.
- A ChatGPT adapter can be active while its insertion-point selector is stale. The tested ChatGPT UI used `[data-composer-leading-controls]`, a localized plus button, and a localized send button `aria-label="Hantar"`; legacy selectors did not locate the current composer controls.
- An expanded SuperAssistant sidebar can overlap the ChatGPT send-button region. Inspect element bounds/hit testing and collapse the sidebar before retrying a blocked send action.
- A tab can exist in CDP without being the active visible window. Confirm target activation and verify the actual desktop screenshot before assuming the GUI is showing that tab.
- A message typed into a composer is not necessarily submitted. Confirm that it appears as a conversation message and that the composer is cleared.

## 5. Xubuntu GUI control — agchat-gui.py

A local helper was created and exercised through RDC:

Executable:
`/home/dzu/AgChat-Testing/session-2/agchat-gui.py`

Documentation created on the Xubuntu host:
`/home/dzu/AgChat-Testing/session-2/AGCHAT-GUI-CONTROL.md`

Screenshot directory:
`/home/dzu/AgChat-Testing/session-2/screenshots/`

Supported helper commands observed:
- `windows` — enumerate desktop windows.
- `screenshot --name <name>` — capture the desktop.
- `focus <window-title>` — focus a window.
- `pointer` — query pointer position.
- `move <x> <y>` — move pointer.
- `click <x> <y>` — click within screen bounds.
- `type <text>` — type text.
- `key <key>` — send a key.
- `atspi` — inspect native accessibility information; `--find <text>` searches exposed labels.

Examples:
```bash
/home/dzu/AgChat-Testing/session-2/agchat-gui.py windows
/home/dzu/AgChat-Testing/session-2/agchat-gui.py screenshot --name current
/home/dzu/AgChat-Testing/session-2/agchat-gui.py atspi --find 'Save'
```

The helper is invoked using the authorised RDC process-start action. It deliberately does not expose arbitrary shell passthrough. Coordinate clicks are bounded to the current screen dimensions.

Underlying utilities observed in the tested setup include X11 screenshot/window/pointer/keyboard/mouse utilities such as `scrot`, `wmctrl`, and `xdotool`, plus AT-SPI2/PyAT-SPI for accessibility discovery.

### GUI control rules

- Take a screenshot before interacting with an unfamiliar or changed screen.
- Enumerate/focus the correct window before sending keyboard or mouse input.
- Use semantic browser DOM controls via CDP where available; use coordinates only when needed.
- Re-screenshot after important actions, especially login, settings, dialogs, navigation, or a potentially destructive click.
- Never assume a click worked. Verify the resulting screen or application state.
- Do not use stale coordinates after changing window size, zoom, sidebar width, or screen resolution.
- AT-SPI2 exposed Chrome as an application and top-level frames, but did not expose the Chromium renderer subtree in the tested session. Use CDP/Puppeteer for web page DOM and extension popovers.

Verified GUI helper smoke checks: full desktop screenshot, window enumeration, pointer query/movement, AT-SPI application enumeration (26 app children observed), Chrome application detection, Python syntax compilation, and screenshot artifact creation all passed.

## 6. SuperAssistant components being configured (not the development method itself)

The tested extension/proxy arrangement used:
- Google Chrome Stable.
- MCP SuperAssistant extension version `0.7.3` in the original installation.
- Local proxy listener at `127.0.0.1:3006`.
- SSE endpoint observed: `http://localhost:3006/sse`.
- Desktop Commander MCP server.

The extension reported Connected and 30 tools in the tested configuration: Desktop Commander 25/25 and Browser 5/5. Actual schemas were retrieved for `desktop-commander.write_file` and `desktop-commander.read_file`; a directory listing, file write/read/edit/read-back, and controlled process execution were demonstrated.

Do not assume the proxy supports every transport endpoint just because it is documented by a client. Confirm the active endpoint and transport from the installed version and live listener.

### Auto controls

The isolated TEST A unpacked extension copy was patched and tested separately from the original Web Store extension.

Verified in the isolated copy:
- MCP toggle.
- Auto Insert toggle.
- Auto Submit toggle.
- Auto Execute toggle.
- Auto Insert and Auto Submit can be toggled independently.
- Final tested state was MCP ON, Auto Insert ON, Auto Submit ON, Auto Execute OFF.
- A fresh read-only call demonstrated manual Run → tool result → Auto Insert → Auto Submit → ChatGPT follow-up.

**Auto Execute must remain OFF** for this working method unless the user explicitly approves a separately scoped test. Tool execution should require deliberate manual Run/approval.

Do not infer Auto Execute from Auto Insert or Auto Submit. These are separate controls and separate behaviours.

## 7. Combining the control planes — preferred workflow

For a task that involves the Xubuntu desktop and browser:

1. **Orient:** Use RDC to confirm host availability and working directory.
2. **Observe:** Capture GUI screenshot and enumerate windows; use CDP to identify the exact browser target.
3. **Inspect:** Use CDP for browser DOM/runtime, AT-SPI for native desktop controls, and RDC for host/process/file state.
4. **Plan:** Choose the narrowest control plane for each action. Avoid using coordinates when a stable DOM selector or dedicated tool exists.
5. **Act:** Execute one meaningful action at a time for UI-sensitive tasks. Use RDC process calls for host operations.
6. **Verify:** Read back the changed state through an independent observation where practical (DOM, screenshot, file content, process output, Git diff, hash).
7. **Recover:** If a result is delayed, inspect the original process/request before retrying. Determine whether the side effect already happened.
8. **Record:** Update the relevant SoT test report and preserve evidence.
9. **Respond:** Give a compact status: outcome, proof, limitation/blocker, next action.

Do not turn the workflow into a fixed sequence of unnecessary tool calls. Select tools based on the actual task and use the minimum number of control surfaces that can reliably observe and perform it.

## 8. Safety, approval and duplicate-execution rules

- Keep Auto Execute OFF.
- Require explicit approval before a write or command with side effects.
- Prefer read-only inspection before mutation.
- Use disposable synthetic workspaces for experiments.
- Protect credentials, personal files, production data and unrelated projects.
- Inspect Git status/diff after source changes.
- Do not assume Desktop Commander `allowedDirectories` confines shell commands; it is not a shell sandbox.
- Avoid destructive disk/system commands, broad deletion, formatting, partitioning, bootloader changes, or credential exposure.
- Do not blindly retry after timeout, lost response, process crash, or missing acknowledgement.
- Treat uncertain side-effect status as `UNKNOWN` until evidence establishes whether it happened.
- Use operation keys/idempotency guards for side-effecting fixtures; distinguish application-level deduplication from transport-level deduplication.
- Never claim exactly-once execution unless the specific side-effect adapter and recovery boundary prove it. Existing tests establish only the explicitly documented fixture guarantees.

## 9. Evidence and documentation standard

For every meaningful test, record:
- Date and environment.
- Exact tool/control plane used (RDC, CDP, GUI helper, MCP bridge, or direct connector).
- Browser, extension, proxy, MCP server and relevant package versions.
- Target path/tab/window and non-secret configuration.
- Intended action and expected result.
- Actual result, including stdout, stderr, exit code, process ID, and response where applicable.
- Independent read-back or screenshot.
- Git status/diff and hashes where relevant.
- Retries, timeouts, duplicate observations, limitations and unresolved risks.
- Result label: PASS, PARTIAL, FAIL, INCONCLUSIVE, or NOT EXECUTED.

A direct RDC result must not be labelled as a proxy E2E pass. A manually activated Run action must not be described as Auto Execute. An inserted draft must not be described as a submitted message.

## 10. Current verified status and next boundary

Verified working capabilities from the recorded sessions:
- RDC access to the Xubuntu host and process/file operations.
- Desktop Commander MCP connection and tool discovery through SuperAssistant/proxy.
- File create/read/edit/read-back and controlled process output through the MCP bridge.
- CDP browser target/DOM/runtime inspection and extension UI troubleshooting.
- Xubuntu GUI screenshot, window, pointer, keyboard/mouse and AT-SPI application inspection through `agchat-gui.py`.
- Isolated TEST A extension popover restoration and independent Auto Insert/Auto Submit toggles.
- UI-originated read-only tool result returning to ChatGPT with Auto Insert and Auto Submit.
- Test 42 real Desktop Commander MCP execution through the direct connector path, with SQLite intent/approval ledger, proof file and duplicate submission suppression.
- Test 43 full E2E route: originating ChatGPT UI → isolated SuperAssistant extension → local SSE proxy → AgChat Orchestrator → Desktop Commander `read_file`, with proof marker and SQLite reconciliation; Auto Execute remained OFF.

Important boundaries:
- Test 42 was a direct Desktop Commander connector call, not ChatGPT → SuperAssistant proxy → Orchestrator → Desktop Commander E2E.
- The original Web Store extension was not modified by the isolated TEST A extension patch.
- Auto Execute remains OFF.
- Proxy transport duplicate suppression is not established by the duplicate JSON-RPC ID probe.
- Test 43 full proxy/orchestrator route is PASS after the originating ChatGPT UI invoked `agchat-orchestrator.agchat_reconcile_test43` through SuperAssistant and the Orchestrator verified the existing proof via Desktop Commander `read_file`. The earlier `start_process` response was misparsed as a failure; reconciliation verified the already-created effect without rerunning the fixture.

## 11. User-facing communication style

The user explicitly prefers compact visible updates to conserve chat space. Apply this format:

```text
Test N — PASS / PARTIAL / FAIL
Bukti: <marker, path, hash or commit>
Batasan: <only if material>
Next: <single next action>
```

Do not dump raw logs unless needed for diagnosis or requested. Keep detailed evidence in the SoT. Concise reporting must never omit necessary verification, approvals, or documentation work.
