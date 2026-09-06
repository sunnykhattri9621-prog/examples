# Ollie's Phonics Adventure

A single-file, zero-dependency phonics tutor for a 6-year-old with no prior
phonics experience. Everything — HTML, CSS, JavaScript, the animated teacher
avatar and the sound effects — lives inside `index.html`. There is no build
step, no bundler and no network request.

## Run it

Open `index.html` in **Chrome** or **Edge**.

For the microphone ("Talk to Teacher") feature, serve the folder rather than
opening the file from disk, because browsers restrict microphone access on
`file://` pages:

```bash
npx serve .        # then open http://localhost:3000
# or
python3 -m http.server 8000
```

If the microphone is unavailable or the grown-up declines the permission
prompt, the app degrades gracefully: Ollie says every sound *with* the child
instead, and the lesson continues exactly as before.

## What it teaches

Single-letter sounds in the Orton-Gillingham / Jolly Phonics order —
**S, A, T, P, I, N** — the first six letters because they blend into many
real words (sat, tap, pin, nip, tin).

Each letter is a three-step lesson, and every step earns a star:

1. **Sound Discovery** — Ollie introduces the letter, stretches the sound out
   loud (`/sssss/`), and shows two big illustrated example words. Tapping a
   picture hears it again.
2. **Listen & Repeat** — Ollie asks the child to make the sound and opens the
   microphone. Recognition is deliberately forgiving.
3. **Sound Tap Game** — "Which one starts with /sssss/?" with two large
   picture tiles.

## Design notes

**No reading required.** Every instruction is spoken aloud. Navigation is
icons only: 🏠 home, 🔁 hear it again, 🎤 my turn, ➡️ next. Touch targets are
large and round, colours are vibrant pastels, and text is only ever a
reinforcement of what Ollie just said.

**The avatar has four states.** Ollie is pure SVG animated with CSS:
*idle* (breathing, blinking), *talking* (bouncing with a moving beak),
*listening* (head tilt, pricked ear tufts, pulsing microphone halo) and
*celebrating* (bouncing with confetti and flying stars).

**Voice.** `speechSynthesis` at `rate: 0.85`, `pitch: 1.2` for a slow, warm,
high teacher voice; phonetic sounds drop to `rate: 0.55` so `/sssss/` really
hisses. An `en` voice is chosen from the browser's list where one exists.

**Forgiving audio matching.** Six-year-olds mumble and recognisers guess whole
words, so a spoken answer is accepted if it is on the letter's accept list
(`"ss"`, `"es"`, `"sun"`...), if any word in it starts with the target sound,
or if it is within a small edit distance of an accepted form. Voicing twins
(p/b, t/d, s/z) are accepted on purpose: a recogniser routinely hears a small
child's /p/ as "b", and a false accept costs a beginner nothing while a false
reject discourages them.

**Zero negative reinforcement.** Nothing is ever marked wrong. An unrecognised
attempt gets "Great try! Let us listen together again", the sound is modelled
once more, and after a second attempt the sound is practised together and the
star is awarded anyway. A wrong tap in the game wobbles gently, re-models the
sound and leaves both tiles live.

**Progress** is kept in `localStorage`, so stars survive a refresh.
