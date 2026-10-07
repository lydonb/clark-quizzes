# Clark's quiz games

Gamified study quizzes for Clark (7th grade, has ADHD), published with GitHub Pages at
https://lydonb.github.io/clark-quizzes/ (public repo, `main` branch, root `/`).
Pushing to `main` redeploys automatically. Only commit/push when asked.

## Layout
- `index.html`: landing page that links to each quiz. Add one `<a class="q">` per new quiz.
- `<subject-slug>/index.html`: one self-contained quiz per folder (e.g. `american-revolution/`).
  Single HTML file, vanilla JS/CSS, **no build step, no external dependencies**, must work offline from `file://`.

## Making a new quiz
The user supplies photos of notes, study guides and textbook pages. Read them closely (handwriting included) and:
1. `mkdir <subject-slug>` and copy `american-revolution/index.html` into it as the starting point. Do not rewrite the engine.
2. Replace only the content and theme:
   - Data banks at the top of the script: `MC`, `TF`, `FB` (fill-in-the-blank), `PAIRS` (term/clue, used by Match-up and Mystery clue), `BRIT`/`COLS` (two-bucket sort), `TIMELINE`. Topics are the `t` field; the home screen has 3 topic buttons plus "Everything" (`acts`, `war`, `time`).
   - If the subject has no timeline or no two-sided sort, delete that generator from `GENS` (see "Generators" below). Rename the topic keys/labels in the home screen if they don't fit.
   - Theme strings: `<title>`, the `<h1>`, subtitle, the enemy name and flag ("Redcoat Army"), `RANKS`, `BADGES`, and the colors in `:root`.
   - **`KEY` (localStorage) must be unique per quiz**, e.g. `clark-<slug>-v1`. All quizzes share one origin, so a reused key would mix up progress.
3. Make it mostly about the user's notes and study guide; use the textbook only for context/extras.
4. Add the quiz to the root `index.html`.
5. Test it: open it in the browser pane and simulate a mission through the UI (answer every question type, right and wrong), check the console for errors, and check that two runs differ.

## Design rules (keep these, they matter for ADHD)
- Short missions (6/10/16 questions), a variety of question types, no timers, no harsh penalties.
- Instant feedback with a one-line explanation; missed questions return in a "Rally round" for partial XP.
- Game feel: XP, streak bonus, enemy HP bar, ranks, badges, confetti/sound (mute toggle, respects reduced-motion).
- Big tap targets, phone/tablet friendly, high contrast, 1-4 / T / F keys and Enter for keyboard play.
- Every run must differ: options are shuffled at render time, distractors/timeline sets/pairs are drawn at random, and `weight()` favors unseen and previously-missed questions.

## Generators (engine overview)
`GENS` maps a type to `{w: weight, heavy?: 1, make(topic, usedIds)}` returning a question object or `null` when the topic can't supply one. `buildQueue` mixes them, never repeats a type back to back, and caps the "heavy" types (order, match, sort). Question types: `mc` (also used by clue/year/first), `tf`, `fill`, `order`, `match`, `sort`. Renderers live in `R`, scoring in `resolve()`.

## Accuracy
Kids' notes often contain mistakes. When the notes are wrong or ambiguous, teach the correct fact and tell the user which items differ so they can check against the teacher's material. Don't silently "fix" the teacher's numbers (e.g. years printed on the study guide).

## Coverage panel
The home screen shows how much of the fixed question bank (`MC`/`TF`/`FB`) he has seen and answered correctly last time, per topic, plus a "Study next" mission of unseen/shaky questions only. It uses `S.seen`, `S.last` and `S.missed` (keyed by question id). Random-set types (order, match, sort, clue, year, first) have `noStat` and are not counted. Keep this when cloning for a new subject; progress is per-browser.

## Notes and gotchas
- **Public site:** the repo is public (free GitHub Pages needs that). Never put the kid's full name, school, photos of his notes or other personal info in any file.
- **Shared storage:** all quizzes share one origin, so every quiz needs a unique `KEY`. Bump the version suffix (`-v2`) if you change the saved-data shape in a way old data would break.
- **Fixed-bank ids:** coverage tracking keys on the question text (`"mc:"+q`, `"tf:"+s`, `"fb:"+s`). Rewording a question resets its history; that's fine, just know it.
- **Testing:** the browser pane may block localStorage on some URLs (the game wraps storage in try/catch, so it still runs). Drive a mission through the DOM with `javascript_exec`, e.g. set `sel={len:10,topic:"all",focus:false}; startMission()` and click through, rather than eyeballing it. Check `buildQueue(16, topic)` returns 16 questions for every topic.
- **Known judgment calls in american-revolution/:** Navigation Acts year is 1650 (his study guide; historians often say 1651), and von Steuben is described as Prussian although his sheet says French. Follow the teacher's materials if the user says they differ.
- **Git:** commit and push only when asked. End commit messages with the Co-Authored-By line from the session's attribution reminder. Pages redeploys within a minute or two of pushing `main`.
