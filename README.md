# Nova Worker — GitHub Pages package

Static docs only. The bot runs on Cloudflare Workers, not on GitHub Pages.

## Files

| File | Role |
|------|------|
| `index.html` | Persian install page (GitHub Pages entry) |
| `INSTALL-FA.md` | Full install notes |
| `nova-worker-channel-fixed.js` | Worker source `v26.21.101-channel-v3` |

## Enable Pages

1. Push this folder to the root of a public repository.
2. Settings → Pages → Deploy from branch `main` / root.
3. Do not commit tokens, `.dev.vars`, or `LOGIN_PASS`.

Deploy the Worker itself in the Cloudflare dashboard or with Wrangler. See `INSTALL-FA.md`.
