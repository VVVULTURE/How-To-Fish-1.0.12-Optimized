# How to Fish 1.0.12 — web port (build 98)

A browser (WebGL) port of **How to Fish** v1.0.12 (Steam app 4001890), rebuilt from the shipped
Windows build and hosted on [Red Portal](https://redportal.dpdns.org).

This repository holds **only the packaged, deployable output**. The Unity project, the shader
reconstruction toolchain, the Steam game files and the working notes are not here — see
`PORT-NOTES.md` in the build workspace for all of that.

## Layout

```
index.html              loader page; tab title carries the build number
local_server.py         serve this folder locally for testing
Build/
  WebGL.data.part1..8   payload, split at 24 MB so every file stays GitHub-safe
  WebGL.wasm.000..002   likewise
  WebGL.framework.js
  WebGL.loader.js
StreamingAssets/        addressables + localization
```

`index.html` reassembles the split parts at load time and appends a cache-busting `?v=<tag>` to
every payload request, so a stale browser cache cannot silently serve an old build. **The tab title
is the build identity** — if it does not read `How to Fish 1.0.12 b98`, the browser is running
something else.

## Running it

```
python local_server.py        # then open the address it prints
```

Any static file server works; all paths in `index.html` are relative, so it can live in a subfolder.

## Multiplayer

The game is peer-hosted: one player's browser acts as the server. Browsers cannot accept incoming
connections, so frames are forwarded by a relay running inside Red Portal's own Node process at
`wss://redportal.dpdns.org/htf-relay`. That URL is baked into `index.html` as `HTF_RELAY_URL`.

* Relay status: <https://redportal.dpdns.org/htf-relay/status>
* Relay source: `htf-relay.js` in `VVVULTURE/red-portal-DKR-LCL`
* If `HTF_RELAY_URL` is empty the game reports *"Multiplayer is unavailable: no relay server is
  configured"* and single-player still works.

## Differences from the Steam build

* Steam friend invites are replaced by 6-digit room codes.
* Anti-aliasing is SMAA rather than TAA.
* Decal rendering layers are off — required, or **no creature renders at all** (see the notes).
* VFX Graph effects (fire, splashes, explosions) do not render on WebGL.
* Menu credits link to <https://github.com/VVVULTURE> and <https://redportal.dpdns.org>.
