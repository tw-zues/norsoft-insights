# Live Leaderboard Backend Plan

The current public game uses local-only scores. A real leaderboard should be implemented as a server-side API, not as client-side secrets in `index.html`.

## Recommended Public Player Fields

Collect enough for bragging rights without overcollecting:

- `display_name`: public name shown on leaderboard
- `handle`: optional LinkedIn/X/company handle
- `company`: optional public company name
- `mode`: `INTERN`, `SDR`, or `FOUNDER`
- `score`
- `day`
- `meetings`
- `certs_passed`
- `knowledge_misses`
- `gap_categories`
- `replay_hash`: optional integrity/debug hash of run summary
- `created_at`

Keep email private. If email is collected for follow-up, do not display it publicly.

## API Shape

Public endpoints:

```text
GET  /api/pipeline-defense/leaderboard?mode=FOUNDER&limit=25
POST /api/pipeline-defense/scores
```

Admin endpoints:

```text
GET    /api/admin/pipeline-defense/scores?status=pending
PATCH  /api/admin/pipeline-defense/scores/:id
DELETE /api/admin/pipeline-defense/scores/:id
```

Suggested score statuses:

- `pending`: submitted but not public yet
- `approved`: visible on public leaderboard
- `hidden`: removed from public view
- `flagged`: suspicious and awaiting review

## Abuse Controls

Minimum controls before launch:

- server-side schema validation
- score/day sanity bounds by mode
- rate limit by IP and user agent
- optional CAPTCHA or Turnstile after repeated submissions
- profanity/slur filter for public display fields
- duplicate-run detection using score summary hash
- moderation queue for top scores or suspicious scores
- admin audit log for edits/deletes

## Admin

Admin editing should live behind authenticated server-side access. Do not ship an admin password, token, or service key in this repo.

Minimum admin actions:

- approve score
- hide score
- edit display name/company/handle
- mark score as suspicious
- delete abusive entry
- export CSV

## Storage

Good lightweight options:

- Supabase table plus Row Level Security and serverless admin actions
- Cloudflare Workers + D1/KV + Cloudflare Access for admin
- Vercel/Netlify functions + Postgres
- Norsoft-owned API/database if this is added to the main website stack

## Client Integration

When the backend exists, the game client should use a public API base URL only:

```js
window.NORSOFT_PIPELINE_DEFENSE_API = "https://example.com/api/pipeline-defense";
```

The client may POST player submissions and fetch public approved scores. It must never contain admin credentials or private database keys.
