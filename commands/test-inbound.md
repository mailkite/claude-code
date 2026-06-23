---
description: Fire a signed test event at a domain's webhook and confirm it's wired up
argument-hint: "[domain]"
---

Verify a domain's inbound webhook is correctly configured by sending it a signed test event.

Steps:
1. Call `mailkite_list_domains`. Choose the domain matching `$ARGUMENTS` if given, otherwise list the domains and ask which one (note each domain's `webhookUrl`).
2. If the chosen domain has **no** webhook set, say so and offer to set one with `mailkite_set_webhook` (ask for the URL first). Don't set one without confirmation.
3. Call `mailkite_test_webhook` for that domain to deliver a signed `email.received` test event.
4. Report the delivery result (status code). If it failed, explain the likely cause (endpoint down, non-2xx, signature not being verified) and point the user at `/mailkite:debug-webhook`.
5. Remind the user how to verify the `x-mailkite-signature` header on their side — they can use the `mailkite_verify_webhook` tool (it runs locally, no network) to check a captured payload.
