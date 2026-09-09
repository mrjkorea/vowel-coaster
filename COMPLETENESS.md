# Completeness — Vowel Coaster (GitHub Pages)

Live site (after `main` publish): https://mrjkorea.github.io/vowel-coaster/  
Pages source: `main` `/` (legacy GitHub Pages). This repo **is** the published Vite `dist`.

## What I checked

Play files (local tree + live HTTP + nested `/vowel-coaster/` sim):

| Path | Role | Live before fix |
|---|---|---|
| `index.html` | Shell, relative `./assets/` | 200 |
| `assets/index-Bco5u1D5.js` | Engine + HUD | 200 |
| `assets/index-O8FwHmoN.css` | Full-viewport layout | 200 |
| `assets/RollerCoaster-DT0xrHnx.js` | Fork rails geometry (dynamic import) | 200 |
| `packs/short-vowels-en.json` | Default JSON pack | 200 at project path; **404 at domain root** |
| `packs/demo-swap-ai.json` | Optional `?pack=` swap | 200 |
| `audio/fish-mimic/*.mp3` | Baked vowel mp3s | 200 |
| `audio/round1/*.mp3` | Demo pack | 200 |
| `audio/youtube-short/*.wav` | Older baked wavs | 200 |
| Pictures / textures folders | Not used | N/A — signs are canvas; park is procedural |
| HOW-TO-PLAY | Missing | **404** |

Headless Chrome against a `/vowel-coaster/` nested static server (same path shape as Pages):

- Pack id `short-vowels-en` loads; no `pre.vc-err`.
- All five item buffers `loadedAudio: pack` (not synth/none).
- Unlock starts **A · E**, two tracks; cracks HUD lists **A E I O U**.
- `trackKind: RollerCoasterGeometry+split-forks`; chase camera on TAP TO RIDE.
- Vowel pulse `times: 1`, `onsets: [0]` (one baked play).
- Keyboard `1` locks a lane. Touch lane buttons render LEFT/RIGHT.
- `HOW-TO-PLAY.html` opens. No `speechSynthesis`, no `/api/tts`.

## What was broken

1. **Absolute pack URL.** Boot used `/packs/short-vowels-en.json`. On Pages that is `https://mrjkorea.github.io/packs/...` (404). The game never mounted the pack, so the ride showed a thrown error (or silent-fail + synth if a host happened to serve `/packs`).
2. **Scream-synth fallback.** Failed audio decode/load scheduled sine oscillators. That is not a vowel recording.
3. **Default pack pointed at 8s YouTube wavs**, not the short fish-mimic mp3s. `fish-a.mp3` is ~66s and `fish-u.mp3` ~14s — unusable as a one-shot cue.
4. **No HOW-TO-PLAY** page or start-screen controls copy.
5. **Phone HUD overlap.** PAUSE/ORBIT sat on the A–U crack chips in portrait.

Gameplay (chase camera, forks, tap + keys, A→E→I→U→O from JSON) was already in the engine. It could not run on the live Pages URL because of (1).

## What I fixed

- `vcPackUrl()` resolves `./packs/...` against the page. A leading `/packs/...` is rewritten to the project folder so GitHub Pages works.
- Default pack audio → short baked fish-mimic mp3s (A `fish2-a`, E `fish2-e`, I `fish-i`, O `fish-o`, U `fish2-u`). Word list stays in JSON.
- Missing buffer → `{source:"none"}` (no oscillator vowel). Mechanical coaster clicks/rumble still use Web Audio FX.
- `HOW-TO-PLAY.md` + `HOW-TO-PLAY.html`, plus start-screen how-to + link.
- Narrow-screen HUD: hide version chip, drop camera/pause buttons below the top chips.

## Remaining gaps

- **Live Pages (`main` `/`) is updated.** Confirmed: engine has `vcPackUrl`, pack 0.12.1 points at fish-mimic mp3s, `HOW-TO-PLAY.html` returns 200.
- No favicon (`/favicon.ico` 404). Does not block play.
- `audio/youtube-short` wavs, long `fish-a` / `fish-u`, round2, and `aipa-short` wavs remain on disk unused by the default pack.
- No `pictures/` or `textures/` trees — not required; nothing 404s for images.
- Demo pack `?pack=./packs/demo-swap-ai.json` is A/I only (by design).
- Ambient chain-lift clicks are still tiny oscillators (not vowel TTS).
