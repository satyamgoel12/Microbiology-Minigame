# Gram Slash 🧫

A Fruit-Ninja-style microbiology minigame, built for a college microorganisms
awareness day. Microbes arc up across the screen — **slash the pathogens, let the
good flora pass**. Every slice teaches one fact about the organism you just hit.

Single file, no build step, no libraries, no external requests. Open `index.html`
and it runs — on a phone, on a projector, or offline once it has been installed.

## How to play

| Thing on screen | What it means | What to do |
|---|---|---|
| **Red danger aura** | Pathogen: *S. aureus*, *E. coli* O157:H7, *Salmonella*, *C. albicans*, Influenza A | Slash it — points, with a combo multiplier for several in one swipe |
| **Green halo + dashed ring** | Beneficial: *Lactobacillus*, *S. cerevisiae*, *Rhizobium* | Leave it alone. Slashing it costs a life and −25 ("you nuked your good flora!") |
| **Violet body** | Gram-positive — crystal violet is retained by the thick peptidoglycan wall | visual/teaching cue only |
| **Pink body** | Gram-negative — decolourised, then counterstained with safranin | visual/teaching cue only |
| **Grey spiky ball** | Influenza A — a virus, so not Gram-stainable at all | slash it, it is a pathogen |
| **Gold-ringed cluster (MRSA)** | The superbug. Each pass strips one resistance ring | needs **three separate swipes**, then pays 60 × combo |
| **Teal bacteriophage** | A virus that only kills bacteria | slash it to **lyse every pathogen on screen** |
| **☣ Biohazard** | The sharps bin | never touch it — it ends the run instantly |

Rules: 3 lives (petri dishes, top right). You lose one for slashing a beneficial,
and one for letting a pathogen fall off the bottom of the plate. A clean streak of
15 pathogens with no mistakes grows a plate back; every 10 gives +75. High score,
top combo and best streak are kept in `localStorage`.

The blade is a swab of **70% IPA** — which is exactly why the game rewards a
targeted wipe over carpet-bombing the whole plate.

## Run it locally

Any static server works (a `file://` open works too, but the service worker —
and therefore offline mode — only registers over `http(s)`):

```bash
python3 -m http.server 8000
# then open http://localhost:8000
```

## Deploy to GitHub Pages

The site is plain static files at the repository root, so Pages needs no build.

**1. Get the files onto `main`**

```bash
git checkout main
git merge claude/compassionate-bardeen-dpnb4a   # or merge the PR on github.com
git push origin main
```

**2. Turn Pages on**

1. Open `https://github.com/satyamgoel12/Microbiology-Minigame` → **Settings** → **Pages**.
2. Under **Build and deployment**, set **Source** to **Deploy from a branch**.
3. Set **Branch** to `main` and the folder to **`/ (root)`**, then **Save**.
4. Wait for the green check in the **Actions** tab (roughly a minute on the first run).

**3. Play it**

```
https://satyamgoel12.github.io/Microbiology-Minigame/
```

Pages serves over HTTPS, so the service worker registers and the game becomes
installable: on Android Chrome, ⋮ → *Add to Home screen*; on iOS Safari, Share →
*Add to Home Screen*. After the first visit it plays **fully offline** — useful when
the venue Wi-Fi gives out mid-event.

**Updating a deployed copy:** bump `CACHE` in `sw.js` (`gramslash-v1` → `-v2`)
whenever you change `index.html`, otherwise already-installed copies keep serving
the old cached build.

## Files

```
index.html             the whole game — markup, CSS, and JS in one file (~50 KB)
sw.js                  service worker: precache + offline-first fetch
manifest.webmanifest   PWA manifest (fullscreen, portrait, icons)
icon-192.png           home-screen icon
icon-512.png           home-screen / maskable icon
```

Total payload ≈ 90 KB. Nothing is fetched from a CDN.

## Tweaking it for your event

Everything worth changing sits near the top of the script in `index.html`:

- **`SPECIES`** — the roster. Copy an entry to add an organism: give it a `kind`
  (`path` / `good` / `bomb` / `phage`), a `gram` (`+`, `-`, or `n` for
  not-stainable), a `shape` (one of the drawing routines in `drawBody`), points,
  and its `facts` array. Facts are picked at random per slice.
- **`difficulty()`** — how fast the ramp goes (time and score based).
- **`pickSpecies()`** — spawn odds for bombs, MRSA, the phage, and the
  pathogen/beneficial split.
- **`STAIN`** — the crystal violet / safranin palette.

## Implementation notes

- HTML5 Canvas 2D, vanilla JS, `requestAnimationFrame` with delta time clamped to
  50 ms so a backgrounded tab cannot warp the physics.
- `devicePixelRatio` capped at 2; canvas resizes off `visualViewport`.
- Microbes, splatter, sliced halves, score popups and shockwaves all come from
  object pools — steady 60 fps with no allocation churn during play.
- Hit testing is segment-vs-circle against each swipe segment, so fast swipes
  cannot tunnel past a microbe between frames. Multi-hit microbes get a 0.34 s
  cooldown so one swipe is one hit.
- Every microbe is drawn procedurally (cocci clusters, rods with flagella and
  fimbriae, budding yeast with pseudohyphae, a spiked virion, an icosahedral
  phage). Slices split the body into two clipped halves that fly apart.
- All sound is synthesised through Web Audio on first touch — no audio files.
- `touch-action:none`, `overscroll-behavior:none` and prevented gesture events
  stop scroll/zoom on mobile; layout respects iOS safe-area insets.
