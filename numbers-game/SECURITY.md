# Pipeline Defense Security Notes

Last reviewed: 2026-07-28

## Current Exposure

`numbers-game` is a static browser game. In the current public build it does not submit scores, emails, initials, or gameplay data to a server. Scores are stored in `localStorage` only.

Current risk is therefore mostly client-side:

- script injection/XSS in the page
- malicious third-party assets
- unsafe service-worker scope or caching
- misleading outbound links
- future mistakes when a live leaderboard is added

## Static Hardening

The page includes a static Content Security Policy:

- `default-src 'self'`
- no external scripts
- no external `connect-src`
- no forms
- no plugins via `object-src 'none'`
- service worker limited to same-origin assets

The game intentionally uses inline CSS and inline JavaScript because it is a single-file arcade demo, so the CSP allows `'unsafe-inline'` for `script-src` and `style-src`. Do not add third-party script tags unless the security model is reviewed again.

## Manual Probe Checklist

Before putting the game on a production website:

- Run the page locally and verify no console errors.
- Test initials/email fields with HTML/script-looking payloads and confirm they render only as text or canvas drawing.
- Confirm no secrets, API keys, admin tokens, or private endpoints are present in source.
- Confirm all external links use `target="_blank"` with `rel="noopener"`.
- Confirm the service worker only caches intended same-origin assets.
- Confirm the production host serves HTTPS.
- If embedding inside another site, set HTTP response headers at the host/CDN level:
  - `Content-Security-Policy`
  - `X-Content-Type-Options: nosniff`
  - `Referrer-Policy: strict-origin-when-cross-origin`
  - `Permissions-Policy` with unused browser features disabled

GitHub Pages cannot set all of those headers directly. A production Norsoft deployment should set them at the web server, CDN, or reverse proxy.

## Live Leaderboard Warning

Do not put leaderboard admin credentials, moderation tokens, database keys, or service-role keys in this static client. A live leaderboard requires a server-side API with rate limiting, validation, abuse controls, and authenticated admin actions.
