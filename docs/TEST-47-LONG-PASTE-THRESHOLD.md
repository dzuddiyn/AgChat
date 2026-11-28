# Test 47 — ChatGPT Composer Long-Paste Threshold

**Result:** PASS — reproducible paste-to-text-attachment threshold identified.  
**Date:** 28 November 2026  
**Environment:** Xubuntu Mini PC; Chrome Stable 158.0.8127.88; existing Rdeeen profile clone; existing AgChat ChatGPT conversation; SuperAssistant extension present; Auto Execute OFF.  
**Conversation:** `https://chatgpt.com/c/6b0a6dd3-8a04-83ec-b94b-71e72d826121`

## Objective

Determine why long instructions pasted into the ChatGPT composer appear as a `Teks ditampal` attachment instead of ordinary inline text, and whether the behaviour is specific to clipboard paste.

## Method

- Used the existing Chrome Rdeeen profile and current conversation; no setup was repeated.
- Focused the ChatGPT composer and pasted harmless ASCII test payloads through the desktop clipboard (X11 clipboard + Ctrl+V).
- Inspected the composer DOM and attachment controls through Chrome DevTools Protocol.
- Checked unique beginning, middle and ending markers where recorded.
- Compared clipboard paste with CDP `Input.insertText` at exactly 10,000 characters.
- Did not send any test payload. Auto Execute remained OFF.
- Removed all test text attachments and cleared the composer after testing.

## Results

| Input method | Payload size | Composer result | Outcome |
|---|---:|---|---|
| Clipboard paste | 116 characters | Inline text | PASS |
| Clipboard paste | 4,000 | Inline text; beginning/middle/end markers present | PASS |
| Clipboard paste | 6,000 | Inline text; beginning/middle/end markers present | PASS |
| Clipboard paste | 8,000 | Inline text; beginning/middle/end markers present | PASS |
| Clipboard paste | 8,191 | Inline text | PASS |
| Clipboard paste | 8,192 | Inline text | PASS |
| Clipboard paste | 8,193 | Inline text | PASS |
| Clipboard paste | 8,500 | Inline text | PASS |
| Clipboard paste | 9,000 | Inline text | PASS |
| Clipboard paste | 9,500 | Inline text | PASS |
| Clipboard paste | 9,999 | Inline text | PASS |
| Clipboard paste | 10,000 | Composer converted to a `Teks ditampal` attachment; editable text area became empty and one attachment-removal control appeared | PASS — threshold reproduced |
| Text insertion (not clipboard paste) | Exactly 10,000 | Inline text; no text attachment created | PASS |

## Finding

The observed behaviour is reproducible at the ChatGPT composer input layer:

- 9,999 ASCII characters pasted: remain inline.
- 10,000 ASCII characters pasted: converted into a `Teks ditampal` attachment.
- Exactly 10,000 characters inserted through the non-clipboard text-input path: remain inline.

This isolates the symptom to **large clipboard-paste handling in the ChatGPT composer**, rather than a general message-length limit. The UI exposes the resulting text as a composer attachment. The evidence does not establish that SuperAssistant creates the attachment; no SuperAssistant tool invocation was needed to reproduce it.

The threshold is established for the tested Chrome/ChatGPT session and ASCII payloads. Do not assume the same character boundary for multibyte Unicode, other browsers, future ChatGPT UI versions, or other clients.

## Operational decision

1. Keep normal short instructions in the composer.
2. For a pasted instruction at or above approximately 10,000 characters, expect ChatGPT to create a text attachment.
3. If the payload must remain inline, split it into smaller chunks below the observed threshold (use a conservative ceiling such as 8,000 characters, not 9,999).
4. For large code, logs or documents, prefer writing the content to a reviewed file and sending a short prompt that references the exact path. Inspect the attachment/file content before any action.
5. Do not treat a `Teks ditampal` attachment as a failure: it is a ChatGPT composer representation. Verify that the attachment is present and complete before asking the model to act on it.
6. Do not change SuperAssistant or the proxy based on this symptom alone.

## Cleanup and safety

- No test payload was submitted to the conversation.
- The temporary text attachments were removed.
- Final verification: composer empty; zero `Alih keluar lampiran teks yang ditampal` controls remaining.
- Auto Execute remained OFF.
- Chrome Rdeeen profile remained in use. No Vivaldi/Firefox interaction.
- No extension, proxy or MCP configuration was modified.

## Limitations

- This test establishes the observed threshold in this session only.
- It does not test large Unicode payloads, multiple-line clipboard formats, or browser/ChatGPT version differences.
- It does not prove a SuperAssistant bug or require a SuperAssistant code change.
