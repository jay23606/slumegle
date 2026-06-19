# slumegle

A minimal Omegle-style random video + text chat. Two strangers who open the same
URL get paired automatically; **Next** drops the current partner and finds another.

Built in the same spirit as [p2p-webcams](https://github.com/jay23606/p2p-webcams)
and [p2p-chat](https://github.com/jay23606/p2p-chat): one HTML file, plain JS, tiny
helper methods, no framework, no build step.

## How it works

- **PeerJS** handles the actual peer-to-peer video and the text `DataConnection` — media and messages never touch a server.
- **Firebase Realtime DB** is used only as a lobby: a waiting peer advertises its PeerJS id under `slumegle/lobby`. The next person to arrive claims that slot (removing it) and calls them. If nobody is waiting, you become the one waiting.
- **Next** removes you from any pairing, sends the partner a `__next__` sentinel so they instantly re-queue, and drops you back into the lobby.
- `onDisconnect()` and a `beforeunload` handler clean up your lobby slot if you close the tab.

## Run it

It's a single static file — just open `index.html`, or serve it anywhere:

```
npx serve .
```

Live (via githack), once pushed:

```
https://raw.githack.com/jay23606/slumegle/main/index.html
```

To test locally, open it in **two tabs** (or two browsers). One waits, the other
matches it.

## Config

The demo points at the same public Firebase URL used by p2p-webcams. To use your
own, replace `databaseURL` in `firebaseConfig` and set the `slumegle/lobby` node to
be world read/write in your Realtime DB rules.

## Notes / ideas

- One-to-one only by design (classic Omegle). Group mode would mean tracking multiple calls like p2p-webcams does.
- No moderation, no accounts, no persistence — strangers are anonymous PeerJS ids.
- Camera + mic permission is required; denying it just shows a "blocked" status.
