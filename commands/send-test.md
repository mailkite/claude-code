---
description: Send a test email through MailKite from one of your verified domains
argument-hint: "[to-address]"
---

Send a test email through MailKite to verify sending works end to end.

Steps:
1. Call `mailkite_list_domains` and pick a **verified** domain to send from (status `verified`). If none are verified, tell the user and stop — they need to add and verify a domain first.
2. Send with `mailkite_send`:
   - `from`: `test@<verified-domain>` (or a sensible mailbox on it)
   - `to`: `$ARGUMENTS` if provided, otherwise ask the user for a recipient
   - `subject`: "MailKite test ✉️"
   - `text`: a short note explaining this is a test from the MailKite plugin, plus the current time
3. Report the returned message id and whether it was accepted. If sending fails, surface the exact API error and the most likely fix (unverified domain, from-address not on a verified domain, plan limit).
