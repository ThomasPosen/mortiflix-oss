# Making a custom pipeline

A pipeline is a recipe for one kind of video: the steps, which ones you review, the error checks a session must pass
before it shows you anything, and the craft rules. This guide is the practical version for this machine. The full
reference is [docs/PIPELINES.md](docs/PIPELINES.md).

## Where things live

| What | Where |
|---|---|
| Mortiflix itself (code, built-in pipelines) | `Developer - Programs and Scripts/_Video Explanations/` (fork: ThomasPosen/mortiflix-oss) |
| The studio (projects, renders, keys, taste) | `Developer - Programs and Scripts/Mortiflix Studio/` |
| **Your pipelines** | `Mortiflix Studio/pipelines/<slug>/`, a private git repo of its own (its README covers the brand-specific ones) |

Your pipelines go in the studio, not in this repo. That keeps the fork clean, so `git pull upstream main` never
conflicts. A studio pipeline with the same slug as a built-in one replaces it. If you write one that would help
other people, copy it into `pipelines/` here and open a pull request upstream.

## 1. Start from the closest pipeline

| You want | Copy |
|---|---|
| A longer explanation with a script | `pipelines/explainer` |
| A short single social video | `pipelines/social-short` |
| A logo animation | `pipelines/logo-sting` |
| A set of slides (Instagram carousel) | `Mortiflix Studio/pipelines/carousel` |

```sh
cd "Developer - Programs and Scripts"
cp -r "Mortiflix Studio/pipelines/carousel" "Mortiflix Studio/pipelines/my-pipeline"
```

## 2. Edit `pipeline.json`

Change `slug` (it must match the folder name), `name` and `description`. Then work through the four parts:

**Intake: what you're asked when you start a project.** Ask only what the studio can't decide well on its own.
The brief step can ask the rest, with defaults. Field types: `text`, `long`, `choice` (with `choices` and a
`default`), `files` (lands in the project's `input/<id>/`). `required: true` blocks the start.

**Steps: the stages and how you review each one.**

| `review` | You get | Use it for |
|---|---|---|
| `questions` | A note and questions with defaults | The brief |
| `document` | Text you comment on by paragraph | Scripts, storyboards, captions |
| `frames` | Images you pin notes on | Style frames, covers |
| `video` | Videos you pin notes on at a moment and a spot | Animatics, finals |
| `audio` | Audio you note at a moment | Music, narration |
| `internal` | Nothing: the session finishes it after its checks | Builds and renders between reviews |

`after` lists what a step waits for. Steps with no path between them run side by side. In `carousel`, the storyboard
and the style frames both wait only for the brief, so they're made in parallel. `work` (`script`, `stills`,
`motion`, `audio`) decides which checks a step runs. `delivers: true` marks the step whose approved files are the
deliverables.

**Checks: the mistakes you never want to point out again.** Write them about *errors* (text off the frame, a wrong
date, the wrong logo, clipping), never about taste. Make `how` concrete: what to look at and what counts as wrong.
Sessions must report every check as `pass`, `fixed` or `n/a`, or the submission is refused. The best checks turn
a brand's real rules (a facts sheet every claim must appear in, which logo goes on which background, who gets
credited) into things a session must verify.

**Status lines:** the short sentences you see while a session works.

## 3. Write `PIPELINE.md`: the craft

Every session reads it first. Keep it short and specific:

1. What it makes and for whom, in two lines.
2. **Core rules** that don't bend (five to eight). Add one every time a mistake happens twice.
3. **Format:** size, frame rate, codec, safe areas. The shared Remotion skill only lists 16:9, 9:16 and 1:1, so
   state any other size explicitly (the carousels set 1080×1350).
4. A table of steps: what happens and exactly what gets submitted.
5. Which shared skills to read (`motion-design`, `remotion-motion`, `voiceover`, `music`, `final-pass`).

Leave studio mechanics out: the gate protocol (`harness/GATES.md`) is given to every session already.

Optionally add a `checklist.md` (a production checklist the session copies and ticks off) and `skills/<name>/` for
skills only this pipeline uses.

## 4. Check it, then try it for free

```sh
cd "Developer - Programs and Scripts/_Video Explanations"
node bin/mortiflix pipelines --studio "../Mortiflix Studio"        # validates every pipeline and lists errors
```

To walk every gate without spending anything, use the demo backend in a scratch studio:

```sh
node bin/mortiflix init --backend demo --studio /tmp/mfx-test
cp -r "../Mortiflix Studio/pipelines/." /tmp/mfx-test/pipelines/
node bin/mortiflix new my-pipeline --backend demo --studio /tmp/mfx-test
node bin/mortiflix run --studio /tmp/mfx-test && node bin/mortiflix review --studio /tmp/mfx-test
```

The demo checks the plumbing (steps, dependencies, review modes, checks), not the craft. For the craft, run a real
project on a small brief and read the session transcript (`state/<id>/sessions/*.jsonl`) and journal.

## 5. Save it

```sh
cd "Developer - Programs and Scripts/Mortiflix Studio/pipelines"
git add -A && git commit -m "my-pipeline: what changed" && git push
```

## Running projects

- **Start the studio:** double-click `Mortiflix Studio/Start Mortiflix.command`. It runs `serve` in Terminal,
  where your `claude` login works. Sessions launched from inside the Claude desktop app are not logged in.
- **Brand pipelines with their own launchers:** see the README in your pipelines repo. A launcher can pre-attach a
  brand kit with repeated `--file field=path` flags, so every project starts with the right files.
- **Anything else:** `node bin/mortiflix new <slug> --studio "../Mortiflix Studio"`, or New in the web studio.

## Things to know

- **Taste vs. brand rules.** `Mortiflix Studio/TASTE.md` applies to *every* video. Put rules for one brand or one
  kind of video in that pipeline's `PIPELINE.md` and checks instead.
- **Learned checks.** When a session proposes a new check after a mistake, approve it with
  `mortiflix checks approve <id>`. It then runs on every future project.
- **Fonts and assets** must be ones you're licensed to use in video. Pipelines load them from the project's
  `input/` folder, never from the web.
- **Updating Mortiflix:** `git pull upstream main` in this repo, then `npm install`, then re-run
  `mortiflix pipelines` to make sure your pipelines still validate.
