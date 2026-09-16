# Gram Slash — working notes

## Every change to the game, without exception

1. Edit `index.html`. The whole game lives in that one file: markup, CSS and JS.
2. **Bump the cache version in `sw.js`** in the same commit — `gramslash-v2` →
   `gramslash-v3`, and so on. The service worker precaches everything, so any
   phone that has already opened the game keeps serving the old build until the
   version string changes. Shipping a change without bumping it means nobody
   sees the change.
3. Commit, then push to `main`. GitHub Pages serves `main` from the repo root,
   so `main` is production.

## Constraints to hold on to

- No libraries, no CDN, no build step, no external network requests at runtime.
  Everything — art, sound, icons — is generated in code or committed as a file.
- **Four gameplay colours, maximum:** violet (Gram-positive body), pink
  (Gram-negative body), green (good / safe / reward), red (danger / kill / loss).
  Greys and white are for UI text only. Do not introduce a fifth hue.
- The audience is a college event crowd, most of them not from a biology
  background. Keep the roster small, the front screen short (a call to action,
  not a manual), and put the teaching in the one-line facts and the end-of-round
  tip instead.
- Mobile first: `touch-action:none`, safe-area insets, capped devicePixelRatio,
  object pools, 60 fps on a mid-range Android.
- Difficulty opens gently for about five seconds so first-timers can read the
  red-glow / green-glow rule, then climbs to full speed by roughly forty seconds.
  Gentle intro, not a slow game.
- The fun facts page is Satyam's own curated content. Edit wording only if he
  asks; it is reachable only after a round ends, never from the start page.
- All audio is synthesised in code (effects and the Korobeiniki music loop).
  Never add an audio file — it would break offline size and the no-assets rule.

## Verifying a change

There is no test suite. Serve the folder (`python3 -m http.server 8000`) and
check on a phone-sized viewport: the run starts slow, microbes arc to roughly
70–80% of screen height, nothing clumps into an unhittable cluster, and the
menu fits on a 320×568 screen without scrolling.
