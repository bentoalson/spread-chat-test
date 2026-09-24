# Spread Chat Test Harness

Test pages for The Spread 2026 live event — evaluating the Minnit chat embed
alongside the event stream. Not a production page.

**Live pages** (GitHub Pages):

- `index.html` — branded event page (Spread design, YouTube player, Minnit chat + SDK)
- `minimal.html` — same layout, no branding
- `mod.html` — moderator console built on the Minnit SDK
  (open as `mod.html?staffkey=YOUR_KEY` to join with moderator rank — key comes
  from Minnit dashboard → Chat Settings → Staff Keys; never commit the key)
- `plain.html` — bare vendor embed snippet, for A/B debugging

`server.js` is only for serving the folder locally (`node server.js` → http://localhost:4173).

## Notes

- The chat is an embedded third-party iframe (`organizations.minnit.chat`). Safari
  blocks its storage cross-site, which breaks joining — the production fix is
  serving the chat from a custom domain (`chat.givebutter.com`) and embedding on
  a givebutter.com page so they're same-site.
- Chat bubble/interior styling lives in the Minnit dashboard (Chat Settings →
  Appearance → Custom CSS), not in these files.
