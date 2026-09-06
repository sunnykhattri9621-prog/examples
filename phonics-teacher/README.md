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

All 26 letter sounds, in synthetic-phonics teaching order rather than
alphabetical order, split into five sets so a beginner meets a handful at a
time:

| Set | Letters | Why these, and why here |
|---|---|---|
| 1 | S A T P I N | The classic first six: they blend into many real words (sat, tap, pin, nip, tin) |
| 2 | C K E H R M | Adds the second most useful consonants and a second vowel |
| 3 | D G O U L F | Completes the short vowels |
| 4 | B J V W | The remaining common consonants |
| 5 | Y Z Q X | The awkward ones, left until last |

**X is taught differently.** It is the one letter a child meets at the *end*
of words, so its whole lesson asks "which one **ends** with /ks/?" — fox, box,
six. **Q** is taught as /kw/ and **C** and **K** share the sound /k/, because
that is what they do.

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

**The sounds are synthesised, not spoken.** This is the most important part
of the app. A text-to-speech engine cannot say a phoneme: ask any voice for
`"nnnn"` and it reads the letter *name* ("en"), and `"puh"` comes back as a
syllable with a vowel stuck on the end. Both teach letter naming, which is
exactly what a phonics beginner must not learn first. So the sounds are built
from raw audio with the Web Audio API instead:

| Family | Letters | How it is made |
|---|---|---|
| Voiceless fricatives | s f h | Shaped noise. /s/ is a 6.8 kHz band with 98% of its energy above 3 kHz |
| Voiced fricatives | z v | The same noise laid over a hum — the hum is the whole difference from /s/ and /f/ |
| Vowels | a e i o u | Formant synthesis: female-speaker F1/F2/F3 over a 190 Hz glottal source |
| Nasals | m n | A hum with the mouth resonances damped away; over 80% of energy below 600 Hz |
| Liquids and glides | l r w y | Voiced resonants. /r/ is defined by its unusually low F3; /w/ and /y/ *glide*, their formants still moving |
| Voiceless plosives | p t k | Closure silence, then a burst. Nothing after it |
| Voiced plosives | b d g | A voice bar through the closure plus a 55 ms murmur — enough to hear, too short to become "buh" |
| Affricate | j | A stop that releases into a fricative |
| Two sounds joined | x q | /k/+/s/ and /k/+/w/ |

Nothing is glued on after a plosive, so /t/ is `/t/` and not "tuh". Because
this is generated audio it also does not depend on which voices a device
happens to have installed. Speech synthesis is still used, at `rate: 0.85`
and `pitch: 1.2`, but only for Ollie's English sentences.

The voice pitch is deliberately 190 Hz rather than something higher and more
cartoonish: at 240 Hz the harmonics are spaced too coarsely to resolve F1, and
/e/ and /i/ came out peaking on the same harmonic — two different vowels that
sounded the same.

All 25 distinct sounds are checked by rendering each one through an
`OfflineAudioContext` and measuring it: that every voiced sound carries more
low-frequency energy than its voiceless twin (b/p, d/t, g/k, v/f, z/s), that
the vowels' F1s stay in order and their F2s land in the right region, that
/r/ shows its low F3 against /l/, that the plosives stay short and the
fricatives stay holdable, and that nothing renders silent.

Tap the big letter (it carries a 🔊 badge) to replay the pure sound as often
as the child wants, and the 🔊 button on the welcome screen plays a test sound
so a grown-up can check the volume before starting. The sounds can be
auditioned from the console with `OlliePhonics.play('s', 3)`.

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

**Pictures lead, words follow.** The printed word under each picture is a
quiet caption for the grown-up. The child is never asked to read anything —
every question is asked out loud and answered by tapping a picture, saying a
sound, or tapping the letter to hear it again.

**Progress** is kept in `localStorage`, so stars survive a refresh.
