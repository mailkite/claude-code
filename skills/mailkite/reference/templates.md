# Email templates

Templates are reusable designs stored on the account. A template row has:

| Field | Notes |
| --- | --- |
| `name` | Required. Human label. |
| `subject` | Default subject line. |
| `html` | Rendered HTML body. |
| `text` | Plain-text alternative (always include one). |
| `json` | Optional block-model source the editor round-trips to `html`. |
| `theme` | Optional theme hint (e.g. `light` / `dark`). |

Returned rows also carry `id` (`tpl_…`), `user_id`, and timestamps.

## CRUD (`/api/templates`, bearer)

```bash
# Create (name required; other fields optional seeds)
curl https://api.mailkite.dev/api/templates \
  -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" \
  -d '{ "name":"Receipt", "subject":"Your receipt", "html":"<p>Thanks!</p>", "text":"Thanks!" }'

curl https://api.mailkite.dev/api/templates            -H "Authorization: Bearer $TOKEN"   # list
curl https://api.mailkite.dev/api/templates/tpl_…      -H "Authorization: Bearer $TOKEN"   # get
curl -X PUT    https://api.mailkite.dev/api/templates/tpl_…  -H "Authorization: Bearer $TOKEN" -d '{ "subject":"New" }'
curl -X DELETE https://api.mailkite.dev/api/templates/tpl_…  -H "Authorization: Bearer $TOKEN"
```

`PUT` is a partial update — omitted fields keep their current value.

## Sending a test of a design

`POST /api/templates/test` renders and sends through your verified sending
domain — handy to preview in a real inbox before saving:

```bash
curl https://api.mailkite.dev/api/templates/test \
  -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" \
  -d '{ "to":"you@example.com", "subject":"Preview", "html":"<p>Preview</p>", "text":"Preview" }'
```

Requires `to` and at least one of `html` / `text`.

## Authoring guidance

- Write a **`text` part for every `html`** — it improves deliverability and
  covers plain-text clients.
- Keep HTML inline-styled and table-based for broad client support; avoid external
  CSS and JS.
- To send a saved template, fetch it, then `POST /v1/send` with its `subject` /
  `html` / `text` (and your `from` on a verified domain). Interpolate any
  per-recipient values before sending.
