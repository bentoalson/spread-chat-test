# Minnit Chat Embed — Website Editor Handoff

The chat is a cross-origin iframe. **Nothing on the page can style anything inside
it** — bubbles, fonts, scrollbars, and interior colors are all configured in the
Minnit dashboard (Chat Settings → Appearance → Custom CSS). The page owns the
frame around the chat, the sizing of the iframe, and the fallback behavior below.
`index.html` in this repo is the working reference implementation.

## 1. The embed snippet

```html
<!-- Minnit chat, SDK-enabled -->
<script src="https://minnit.chat/js/chatsdk.js?c=1772345196" defer></script>
<script src="https://minnit.chat/js/embed.js?c=1772345192" defer></script><span
  style="display: none;"
  class="minnit-chat-sembed"
  data-iframeid="chat"
  data-chatname="https://organizations.minnit.chat/477805246003388/c/Main?embed&sdk&sdkversion=1.1"
  data-style="width:100%; height:100%;"
  data-version="1.55">Chat</span>
```

Notes:
- Keep the **script embed method** (the `<span class="minnit-chat-sembed">` +
  `embed.js`), not a raw `<iframe>`. The script method stores session data in the
  host page's localStorage, which matters for browsers that block third-party
  cookies.
- Keep `data-style` at `width:100%; height:100%` and size the chat with a page
  container instead (next section). Anything else here (fixed px, min-height)
  gets written as inline styles on the iframe and fights the layout.
- The `&sdk&sdkversion=1.1` parameter + `chatsdk.js` enable the JS in section 3.
  If a page needs no SDK features, both can be dropped and the snippet still works.

## 2. Sizing CSS the page MUST provide (not handled by Minnit)

Minnit's `embed.js` measures the container once at inject time and writes pixel
dimensions onto the iframe. On responsive layouts that goes stale (wrong width
after resize, iframe overflowing its box). Pin the iframe to its container:

```css
.chat-embed { position: relative; /* size this box however the design wants */ }
.chat-embed .minnit-chat-sembed { display: block; height: 100%; }
.chat-embed iframe {
  position: absolute;
  inset: 0;
  width: 100% !important;   /* beats the inline styles embed.js writes */
  height: 100% !important;
  min-height: 0 !important;
  border: none;
  display: block;
}
```

(`!important` is deliberate — it overrides the inline styles the embed script
sets. Without it the iframe keeps whatever size it measured on first paint.)

## 3. SDK bootstrap (interface icon color + failure fallback)

Two things are handled in page JS, not in the dashboard:

```html
<script>
  document.addEventListener('DOMContentLoaded', () => {
    if (typeof MinnitChat === 'undefined') return;
    const minnit = new MinnitChat({ iframe: document.getElementById('chat') });
    minnit.connect();

    // If no chat session materializes within 8s (e.g. Safari third-party
    // storage failure), reveal a fallback link instead of a stuck loader.
    let connected = false;
    setTimeout(() => {
      if (!connected) document.getElementById('chat-fallback').hidden = false;
    }, 8000);

    minnit.on('connected', () => {
      connected = true;
      document.getElementById('chat-fallback').hidden = true;
      // Interface icons/controls (top bar, send button) follow the accent color.
      // Ink = black icons. Bubble styling itself lives in dashboard Custom CSS.
      minnit.customizeAppearance({ theme: 'bright', accentcolor: '#121926' });
    });
  });
</script>
```

Fallback element (style to taste; keep it OUTSIDE the iframe area so it never
covers the message input — in the reference page it's a footer row of the chat
panel, not an overlay):

```html
<div id="chat-fallback" hidden>
  Trouble with chat?
  <a href="https://organizations.minnit.chat/477805246003388/c/Main"
     target="_blank" rel="noopener">Open it in a new window ↗</a>
</div>
```

A visible "popout" link to that same URL somewhere in the chat header is also
recommended — it's the guaranteed path into the chat for any browser.

## 4. Division of responsibilities

| Concern | Where it lives |
|---|---|
| Frame, borders, extrusions, headers, page layout | Page CSS (the editor) |
| Iframe sizing + overflow control | Page CSS — section 2 above (required) |
| Interface icon/accent color | Page JS — `customizeAppearance` (section 3) |
| Load-failure fallback + popout link | Page HTML/JS — section 3 |
| Bubble shape/colors, interior fonts, scrollbar | Minnit dashboard → Custom CSS |
| Nickname requirement, guest avatar, filters | Minnit dashboard → Guests / Bot |
| Moderator access | Minnit staff keys or mod-ranked accounts |

## 5. Safari (important until the custom domain exists)

On Safari (macOS + iOS), third-party storage blocking breaks joining the embedded
chat — users see "Tap or click to enter" and can land on a blank loader. The
fallback in section 3 covers it short-term. The real fix: serve the chat from a
custom domain (`chat.givebutter.com`, Minnit Lite plan+) and embed it on a
givebutter.com page, making it same-site. Re-test Safari the moment that's live —
it can only be tested from a page actually hosted on givebutter.com.

## 6. YouTube embed (if the editor also places the player)

Use `referrerpolicy="origin"` on the YouTube iframe. Embeds without a referrer
(e.g. opened from local files, or aggressive privacy settings) fail with
YouTube "Error 153".
