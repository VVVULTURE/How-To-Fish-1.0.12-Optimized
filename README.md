# How to Fish 1.0.12 — web port (build 119)

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

`index.html` carries the exact part list, reassembles the parts at load time and appends a
cache-busting `?v=<version>-<content hash>` to every payload, loader and framework request, so a stale browser cache cannot silently serve an old build. **The tab title
is the build identity** — if it does not read `How to Fish 1.0.12 b119`, the browser is running
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
* Sounds are preloaded at startup (except long music/ambience tracks), because WebGL loads audio
  asynchronously and would otherwise drop the first play of every sound. Costs ~400 MB of browser memory.
* Default look sensitivity is 6.00 (Steam: 1.00).
* No microphone / proximity chat.
* The pause menu's Quit saves and returns to the main menu (a browser tab cannot quit).
* Reel of Fortune machine bodies render with a small depth offset (WebGL depth precision).
* "Join the Discord" opens the Red Portal Discord server.
* Menu credits link to <https://github.com/VVVULTURE> and <https://redportal.dpdns.org>.

## Build 119 changes (from 109)

* **Fixed the Reel of Fortune screen turning flat yellow / red / glitchy at a distance or an angle.**
  The machine model has a raised screen plate 3 mm behind the screen's UI. Steam (D3D11, reversed-Z
  float depth) separates them; WebGL's 24-bit depth with the game's 1 cm near plane does not, so the
  plate showed through. The machine bodies now get a small depth offset (1 x slope + 8 depth steps) via
  a material copy; geometry and every other material are unchanged. All five islands' machines.
* "Join the Discord" now opens https://discord.gg/TzEsEJgtJp (the Red Portal server).

## Build 109 changes (from 103)

* **Island ground is grey again, and lightmapped surfaces are lit like Steam.** Two causes:
  lightmap shader variants were stripped at build time (Lightmap Modes: Automatic sees no baked
  data, because the port attaches the Steam lightmaps at runtime), so no lightmap was ever sampled;
  and 67 textures Steam ships uncompressed (the colour palette, UI, font atlases, SMAA lookup) were
  block-compressed, which bled the palette's grey into the magenta beside it.
* Microphone / proximity chat options, Push To Talk binds and the HUD mic meter are removed
  (no mic support on the web).
* Verified through the Google-Sites launcher -> Red Portal blob tab -> game blob tab chain.

## Build 103 changes (from 98)

* **Fixed: the game crashed (stack overflow) whenever the Spider Crab, Giant Piranha, Bowhead Whale
  or Mutated Bowhead Whale boss appeared.** A decompiler defect turned a non-virtual base call into
  infinite recursion. Found by comparing the whole game assembly's IL with the Steam build.
* Fixed: after a multiplayer host returned to the menu, a destroyed player kept running every
  network tick (a NullReferenceException 60 times a second).
* Fixed: the first play of every sound was silent.
* Fixed: settings changed from the main menu were lost when the tab closed.
* Fixed: pause-menu Quit froze the tab.
* Fixed: cache-busting never applied to the deployed page (stale builds could be served), and the
  loader probed for missing files (17 failed requests per load, now 0).
* Default sensitivity 6.00; relay transport hardened against re-entrant stops.
