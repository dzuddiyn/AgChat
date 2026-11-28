# TEST 45 — Workspace filesystem boundary check

**Result: PASS for Desktop Commander filesystem tools.**  
**Date:** 28 November 2026  
**Route:** originating ChatGPT conversation → SuperAssistant → local SSE proxy → Desktop Commander MCP.  
**Browser:** Google Chrome Rdeeen profile clone only.  
**Approval:** Auto Execute OFF; the single tool card was manually run.

## Objective

Verify that Desktop Commander's filesystem allow-list denies a read-only directory listing outside the AgChat disposable workspace.

## Test action

One call only:

```text
desktop-commander.list_directory
path = /home/dzu/Documents
```

The configured allowed directory was:

```text
/home/dzu/AgChat-Testing/session-2
```

## Observed result

The original ChatGPT conversation received this MCP error:

```text
Error: Path not allowed: /home/dzu/Documents.
Must be within one of these directories:
/home/dzu/AgChat-Testing/session-2
```

The tool call completed in approximately 50 ms. No file was read or modified. The request was not retried and no alternate path was attempted.

## Conclusion

**PASS:** Desktop Commander's filesystem allow-list blocks this out-of-scope directory-list operation through the real SuperAssistant proxy path.

**Important limitation:** This does not sandbox arbitrary shell/process execution. A process launched by Desktop Commander may still inherit the Xubuntu user's OS permissions. The allow-list is a filesystem-tool boundary, not a complete host security boundary.

The accepted AgChat PoC direction permits understood, non-destructive host commands on the dedicated development PC with explicit manual approval and sensible safeguards. A container or VM is not a mandatory V1 gate. Stronger isolation remains an optional future improvement if a concrete workload or risk justifies it.

## Safety and evidence

- Read-only operation only.
- No out-of-scope file content was returned.
- No side effect occurred.
- Auto Execute remained OFF.
- One visible Run button was confirmed and manually activated.
- Same originating ChatGPT conversation and existing Chrome Rdeeen profile were reused.
