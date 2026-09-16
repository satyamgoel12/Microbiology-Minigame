# Gram Slash 🧫

A Fruit-Ninja-style microbiology minigame, built for a college microorganisms
awareness day. Microbes arc up across the screen — **slash the harmful ones, let
the helpful ones live**. Every slice shows one line about the microbe you just
hit, and every round ends with one piece of advice worth taking home.

Single file, no build step, no libraries, no external requests. Open
`index.html` and it runs — on a phone, on a projector, or offline once it has
been opened.

**Made by Satyam.**

## How to play

The whole rule is the glow, not the shape:

| On screen | Meaning | Do |
|---|---|---|
| **Red glow** | Harmful: *S. aureus*, *E. coli*, *C. albicans* | Slash it. Several in one swipe multiplies the score |
| **Green glow + dashed ring** | Helpful: *Lactobacillus*, baker's yeast, *Rhizobium* | Leave it alone |
| **Red rings, bigger cluster** | MRSA superbug | Three separate swipes to clear it, worth 60 |
| **Green phage** | Bacteriophage — a virus that kills only bacteria | Slash it to wipe every harmful microbe on screen |
| **☣ Biohazard** | The sharps bin | Never touch it. It ends the round instantly |

Each microbe comes as a matched pair — the harmful rod (*E. coli*) and the
helpful rod (*Lactobacillus*) look alike, as do the two yeasts. That is the
point: shape does not tell you who the enemy is.

Body colour teaches the Gram stain in passing: **violet = Gram-positive**
(the thick wall keeps the crystal violet), **pink = Gram-negative** (it loses
the violet and takes the pink counterstain).

**Lives.** You start with 5 plates. Slashing a helpful microbe, or letting a
harmful one fall off the bottom, costs **one plate**. Every **5 harmful microbes
in a row with no mistakes gives a plate back** (or +50 points if you are already
full). Score, top combo and best streak are kept in `localStorage`.

**Pace.** A round opens gently — one microbe at a time on a long, lazy arc — for
about five seconds, then climbs steadily to full speed by roughly forty seconds,
with up to seven microbes in the air at once. Microbes spawn into four fixed
lanes so they never arrive as one unhittable clump.

**Pause and sound.** The ❙❙ button in the HUD freezes the round; switching tabs or
apps pauses it for you, so nobody loses a life to a phone notification. The ♪
button — on the start page, in the HUD and on the pause screen — turns *all*
sound on or off: the background music and the slice effects together. The choice
is remembered between visits.

**Music.** The loop is *Korobeiniki*, the 19th-century Russian folk tune everyone
knows from Tetris, synthesised note by note in Web Audio as a soft piano-ish
melody with a bass line. There is no audio file, so it adds nothing to the
download and works offline.

**The fun facts page.** When a round ends you can open **Microbial Facts & Health
Guide** — eight curated cards on fibre and the gut, finishing an antibiotic
course, soap versus alcohol, live fermented foods, the hygiene hypothesis, phage
therapy, flossing and the overnight fasting window. It is only reachable from the
end of a round, which is the point: play first, read after.

## Run it locally

Any static server works (opening the file directly works too, but the service
worker — and therefore offline play — only registers over `http(s)`):

```bash
python3 -m http.server 8000
# then open http://localhost:8000
```

## Deploy to GitHub Pages

The site is plain static files at the repository root, so Pages needs no build.

1. Push to `main` (Pages serves `main`).
2. Open `https://github.com/satyamgoel12/Microbiology-Minigame` → **Settings** → **Pages**.
3. **Source:** Deploy from a branch. **Branch:** `main`, folder **`/ (root)`**. Save.
4. Wait for the green check in the **Actions** tab (about a minute on the first run).

It goes live at:

```
https://satyamgoel12.github.io/Microbiology-Minigame/
```

Pages serves over HTTPS, so the game is installable: Android Chrome ⋮ → *Add to
Home screen*, iOS Safari Share → *Add to Home Screen*. After the first visit it
plays **fully offline** — useful when the venue Wi-Fi gives out mid-event.

> **Every time you change the game, bump `CACHE` in `sw.js`** (`gramslash-v2` →
> `gramslash-v3`). Phones that already opened the game keep serving the old
> cached build until that string changes. See `CLAUDE.md`.

## Files

```
index.html             the whole game — markup, CSS and JS in one file (~64 KB)
sw.js                  service worker: precache + offline-first fetch
manifest.webmanifest   PWA manifest (fullscreen, portrait, icons)
icon-192.png           home-screen icon
icon-512.png           home-screen / maskable icon
CLAUDE.md              working rules for anyone (or any agent) editing this
```

Total payload ≈ 105 KB, nothing fetched from a CDN.

## Tweaking it for your event

Everything worth changing sits near the top of the script in `index.html`:

- **`SPECIES`** — the roster. Copy an entry to add a microbe: `kind`
  (`path` / `good` / `bomb` / `phage`), `gram` (`+` or `-`), a `shape` from the
  drawing routines in `drawBody`, points, and a `facts` array (one is picked at
  random per slice). Keep the list short — the crowd is mostly non-biology.
- **`TIPS`** — the one-line advice on the end-of-round screen.
- **`FACTS`** — the cards on the fun facts page (icon, badge, title, body).
- **`MELODY` / `BASS` / `EIGHTH`** — the music. `EIGHTH` sets the tempo; the two
  arrays are `[frequency, length in eighth notes]` pairs.
- **`difficulty()`**, **`spawnWave()`**, **`gravityNow()`** — pace, how many
  microbes share the screen, and how fast they fall.
- **`LANES`** — the horizontal spawn lanes that stop microbes clumping.
- **`MAX_LIVES`** — how many plates a player starts with.
- **`STAIN`** — the crystal violet / safranin palette.

## Implementation notes

- HTML5 Canvas 2D, vanilla JS, `requestAnimationFrame` with delta time clamped
  to 50 ms so a backgrounded tab cannot warp the physics.
- `devicePixelRatio` capped at 2; the canvas resizes off `visualViewport`.
- Spawn velocity is solved from the target apex, so microbes reach 70–80% of
  screen height on any device instead of dying out halfway up a laptop screen.
- Microbes, splatter, sliced halves, score popups and shockwaves all come from
  object pools — 60 fps with no allocation churn during play.
- Hit testing is segment-vs-circle against each swipe segment, so a fast swipe
  cannot tunnel past a microbe between frames. MRSA gets a 0.34 s cooldown so
  one swipe is one hit.
- Every microbe is drawn procedurally — cocci clusters, rods, budding yeast with
  invasive tubes, an icosahedral phage. Slices split the body into two clipped
  halves that fly apart.
- All sound is synthesised through Web Audio on first touch — no audio files.
  The music runs on a lookahead scheduler (notes queued 0.35 s ahead on a 60 ms
  timer) so it stays in time without competing with the render loop, and it stops
  cleanly on pause, game over and mute.
- The round pauses itself on `visibilitychange` and on window blur, so a
  backgrounded tab can never cost a life.
- `touch-action:none`, `overscroll-behavior:none` and prevented gesture events
  stop scroll and zoom on mobile; the layout respects iOS safe-area insets and
  fits a 320×568 screen.
