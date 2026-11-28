# SuperAssistant / ChatGPT Long-Message Handling

**Status:** CLOSED — Test 47 PASS; threshold and practical workaround established.  
**Observed:** 28 November 2026, TEST 46 follow-up in the originating ChatGPT conversation.  
**Evidence:** User screenshot shows a long pasted instruction appearing in the composer as a `Teks ditampal` attachment/chip rather than ordinary inline text.  
**Scope:** Determine whether this is ChatGPT composer paste handling, SuperAssistant behaviour, browser/extension interaction, or a combination. Do not assume the attachment is created by SuperAssistant.

## 1. Why this matters

Long instructions, shell commands and tool payloads may be represented as text-file attachments. This can change how the message is delivered and interpreted, make the visible prompt harder to review, and complicate reproducible same-chat tool execution.

This is a workflow reliability/UX issue. It does not invalidate TEST A's completed MCP route or its acceptance result.

## 2. Current evidence and unknowns

**Observed fact**
- A long pasted instruction was displayed as a `Teks ditampal` attachment in the ChatGPT composer.
- The visible attachment preview showed truncated function-result text.

**Not yet established**
- The character/byte threshold that triggers this behaviour.
- Whether the conversion is performed by ChatGPT's composer, SuperAssistant, or browser clipboard/paste handling.
- Whether the attachment's complete contents are delivered to the model and MCP tools without truncation or alteration.
- Whether the behaviour differs for typed text, clipboard paste, code blocks, or messages containing long commands.

Do not describe this as a SuperAssistant defect until the responsible layer is identified.

## 3. Diagnostic test matrix

Use the existing Chrome Stable Rdeeen profile and the existing originating ChatGPT conversation. Do not repeat browser or MCP setup. Keep Auto Execute OFF.

Test with harmless, uniquely identifiable payloads only:
1. Short plain text typed into the composer.
2. Same short text pasted from clipboard.
3. Medium plain-text paste.
4. Long plain-text paste.
5. A long code/command block pasted as text.
6. A long instruction with a small number of explicit steps.

For each case record:
- Approximate character and byte count.
- Whether the composer shows inline text or a `Teks ditampal` attachment.
- Whether a file/attachment chip appears before sending.
- What the submitted conversation message visibly contains.
- Whether the receiving model/tool can accurately identify a unique marker near the beginning, middle and end.
- Whether the payload is complete, truncated, duplicated or transformed.

Change one variable at a time. Do not send real secrets or destructive commands as test content. Do not run any command embedded in a test payload.

## 4. Interim operating rules

Until the cause and threshold are known:
- Keep tool instructions concise and task-focused.
- Prefer one tool call per instruction where practical.
- For large code or document payloads, write/read the content through the approved file tools rather than pasting a huge block into the composer.
- For long shell work, use a short instruction that references a reviewed script/file path; inspect the script before execution.
- If ChatGPT shows a text attachment, inspect its name/type and verify the submitted message before asking the model to act on its contents.
- Never assume a preview represents the complete attachment.
- Do not blindly resend a long payload after an ambiguous send; inspect the conversation and attachment state first.

These are temporary reliability practices, not a claim that all long messages must be converted to files.

## 5. Exit criteria

This investigation can be closed when:
1. The responsible layer is identified or the behaviour is narrowed to a reproducible boundary.
2. A practical safe input method is selected for short, medium and long payloads.
3. Start/middle/end marker checks establish whether content arrives intact.
4. The working method and AGENTS instructions are updated with evidence-based guidance.

## 6. Test outcome and decision

The investigation is closed by [Test 47](TEST-47-LONG-PASTE-THRESHOLD.md). In the tested Chrome/ChatGPT session, 9,999 ASCII characters pasted remain inline, while 10,000 pasted characters become a `Teks ditampal` attachment. Exactly 10,000 characters inserted through the non-clipboard text-input path remain inline. This points to ChatGPT composer large-paste handling and does not establish a SuperAssistant defect. Use an 8,000-character conservative ceiling for inline clipboard paste; use reviewed files or split payloads for larger content.

## 7. Decision boundary

Do not modify the SuperAssistant extension, ChatGPT browser state, or proxy solely on the basis of the screenshot. First reproduce and isolate the behaviour. Preserve the current passing TEST A setup.
