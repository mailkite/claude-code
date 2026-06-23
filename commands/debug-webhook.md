---
description: Investigate why a MailKite webhook delivery failed and propose a fix
argument-hint: "[message-id]"
---

Help the user debug a failing inbound webhook delivery.

Steps:
1. If `$ARGUMENTS` is a message id, call `mailkite_get_message` for it. Otherwise call `mailkite_list_messages`, summarize the most recent messages, and ask which one to investigate.
2. Inspect the message's deliveries: look at the delivery `status`, `attempts`, and `last_status_code`. Identify whether the failure is delivery-side (endpoint returned non-2xx / unreachable) or signature-side (the user's server rejected a valid signature).
3. Explain the root cause in plain terms and recommend the concrete fix:
   - Non-2xx / timeout → the user's endpoint; check it returns 200 quickly.
   - Signature mismatch → they must verify over the **raw** body using `t` + `v1` from `x-mailkite-signature` (HMAC-SHA256 of `"${t}.${rawBody}"`); offer the `mailkite_verify_webhook` tool to test a captured payload locally.
4. If the endpoint is now healthy, offer to re-drive the delivery with `mailkite_retry_delivery` (confirm first).
