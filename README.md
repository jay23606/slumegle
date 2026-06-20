# slumegle

**[Try it live](https://raw.githack.com/jay23606/p2p-webcams/main/index.html)**

A minimal Omegle-style random video + text chat. Two strangers who open the same
URL get paired automatically; one button cycles you to the next stranger.

Built in the same spirit as [p2p-webcams](https://github.com/jay23606/p2p-webcams)
and [p2p-chat](https://github.com/jay23606/p2p-chat): plain JS, a few tiny helper
methods, no framework, no build step. The whole app is a single `index.html` plus a
PWA manifest, service worker, and icons.

## Features

- **Random one-to-one video + audio pairing** between strangers.
- **Single Start / Next button** — Start joins the pool; the same button becomes Next to skip to a new stranger. **Esc** is a keyboard shortcut for Next.
- **Stop** leaves entirely and resets the button to Start.
- **Live "N online" count** of everyone currently in the app, updated in realtime.
- **Text chat** overlaying the video as a translucent panel, **hidden by default** (most people just want video). A **Chat** button toggles it and flashes an unread indicator when a message arrives while it's closed. Chat history clears on each new pairing.
- **Connection status** feedback (connecting / connected / unstable / failed), a **"still looking…"** nudge if no match appears within 30s, and a **"Stranger is typing…"** indicator.
- **Country tag** — each side shows the other's country with a flag emoji (e.g. "🇩🇪 Germany"), looked up from the user's IP. Country-level only by design; city/region are deliberately not used.
- **Screen sharing** — a **Share** button swaps your outgoing camera for your screen mid-call (no reconnection), and reverts via the in-app button or the browser's native stop-sharing control.
- **Volume slider** overlaid on the video, revealed on hover (always visible on touch devices).
- **Click-to-swap video** — tap the small self-view to make it fullscreen and shrink the stranger into the corner; tap again to swap back. Resets each new pairing.
- **Light / dark theme** toggle (🌙/☀️) that switches the whole page including the area behind the videos.
- **Mobile-friendly** layout that reflows for portrait and landscape; the button bar wraps on narrow screens. Video uses `contain`, so the full camera frame is always shown (letterboxed rather than cropped, never clipped).
- **Installable PWA** — add to home screen for a standalone, full-screen app with an Omega (Ω) icon.

## How it works

- **PeerJS** handles the peer-to-peer video, audio, and the text `DataConnection` — media and messages flow directly between the two browsers, never through a server. The data channel uses JSON serialization (avoids a PeerJS binary-packer bug).
- **Firebase Realtime DB** is used only for coordination, in two small nodes:
  - `slumegle/lobby` — peers currently *waiting* to be matched. A waiting peer advertises its PeerJS id; the next arrival claims that slot (removing it) and calls them. If nobody is waiting, you become the one waiting.
  - `slumegle/online` — everyone currently present, used only for the live count.
- **Signalling between the pair** (next, typing, country) rides the existing PeerJS data channel as small `__sentinel__` messages, not Firebase.
- `onDisconnect()` plus a `beforeunload` handler clean up your lobby and presence entries if you close the tab or lose connection.
- **Country lookup** tries several free IP-geolocation providers in turn (ipwho.is, ipapi.co, geojs.io) so one being down or rate-limited falls back to the next. It is best-effort: VPNs show the VPN's country, and if every provider fails the tag is simply omitted.

## Files

| File | Purpose |
|------|---------|
| `index.html` | The entire app (HTML, CSS, JS in one file). |
| `manifest.json` | PWA metadata (name, icons, display mode, colors). |
| `sw.js` | Service worker — enables install; network-first so the live app and signaling are never stale. |
| `icon-192.png`, `icon-512.png` | App icons (white Ω on black). |
| `icon-maskable-512.png` | Maskable icon with safe-zone padding for Android. |

All files must sit together in the same directory at the site root.

## Run it

It's a static site — serve the folder:

```
npx serve .
```

**HTTPS is required** in practice: browsers block camera/mic on plain `http://`, and
PWA install / service workers only work over HTTPS (or `localhost`). GitHub Pages,
githack, and Netlify all provide HTTPS.

Live (via githack): **[raw.githack.com/jay23606/p2p-webcams](https://raw.githack.com/jay23606/p2p-webcams/main/index.html)**

To test locally, open it in **two tabs** or two browsers. One waits, the other
matches it. Each tab needs its own camera/mic permission.

## Config

The demo points at the same public Firebase URL used by p2p-webcams. To run your own,
replace `databaseURL` in `firebaseConfig` and make the `slumegle` node world
read/write in your Realtime DB rules:

```json
{
  "rules": {
    "slumegle": { ".read": true, ".write": true }
  }
}
```

## Limitations / notes

- **One-to-one only**, by design (classic Omegle). Group mode would mean tracking multiple simultaneous calls.
- **No persistent identity.** PeerJS assigns a fresh random id each page load — no accounts, no stable way to recognize a returning stranger. A blocking feature, for instance, could only be session-scoped without real backend identity.
- **No moderation, no age gate.** This is a demo. An unmoderated public stranger-video service carries real safety and abuse obligations — think hard before pointing it at the open internet.
- **No TURN server.** Connections rely on default STUN, so two users behind strict/symmetric NATs may match but fail to see each other. Adding a TURN relay (Twilio, Metered, self-hosted coturn) fixes that.
- **Screen sharing is desktop-only** — `getDisplayMedia()` is unsupported on most mobile browsers, and sharing your screen with a stranger is a privacy footgun worth a confirmation prompt before going public.
- **IP-geolocation free tiers are rate-limited.** Fine for testing and modest use; real volume needs a keyed provider.
- The matchmaker reads the whole lobby per search — fine for small scale, but it can get racy if the app ever gets popular.
