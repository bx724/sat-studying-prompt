# SAT Command Center

This repository contains the prompt and instructions to allow you to create your own tailored SAT studying dashboard. Please read this README before using the prompt.

The prompt turns an AI coding agent such as [Claude Code](https://claude.com/claude-code) or [Codex](https://openai.com/codex/) into a study system for the digital SAT. It plans your schedule, keeps an error log of every question you miss, builds practice sets from the official College Board question bank, and keeps a dashboard up to date with a projected score.


**See it first:** download [`dashboard.html`](dashboard.html) and open it in your browser. It's a sample dashboard filled with a made-up student's data, so you can see what you'll get before you start.

## How to use it

You don't need to clone this repo. Everything is in one file.

1. **Make a new, empty folder** on your computer for your SAT prep. The agent will create all of your study files in it, so don't use a folder that already has other things in it.
2. **Copy [`PROMPT.md`](PROMPT.md)** from this repo and paste it into a coding agent such as Claude Code or Codex, running in that folder.
3. **Answer the agent's questions.** The prompt asks for everything it needs, so you don't have to prepare anything or edit the prompt.
4. **Open `dashboard.html`** in your browser once the agent has built it.

## What the prompt will ask you

The agent will ask for:

- Your test date
- Your most recent or baseline scores (Reading and Writing, and Math)
- Your target score
- How many hours a week you can study, and on which days
- **A past practice test**

### Uploading a past practice test

**It is highly recommended, but not required.** You can upload a score report, a screenshot of your results, or a list of the questions you missed from a Bluebook practice test. The agent uses it to:

- Set your starting projected score from a real result instead of a guess
- Fill your error log with the questions you actually missed
- Build your first practice sets around your weakest areas

If you skip it, the agent works from the scores you type in, and the plan becomes more accurate as you log practice.

## What the prompt creates

Once you've answered the questions, the agent sets up your folder like this:

```
your-sat-folder/
├── 00-README.md          # index of the folder, and the agent's notes on where you stand
├── 01-plan.md            # your day-by-day schedule through test day
├── 02-error-log.md       # every question you miss, with why you missed it
├── 03-grammar-rules.md   # the Standard English Conventions rules, in your own words
├── 04-score-tracker.md   # every full practice test and its domain bands
├── 05-vocabulary.md      # words in context you didn't know
├── 06-test-day.md        # timing strategy and a logistics checklist
├── 07-desmos.md          # when and how to use the built-in graphing calculator
├── dashboard.html        # your dashboard: calendar, misses, topics, vocab, projected score
├── bank/                 # College Board questions, downloaded once and cached here
└── sets/                 # your practice sets, one PDF per set with an answer key
```

**`sets/` starts empty.** No practice sets are made up front. The agent builds each set when you ask for it, so every set is based on what your error log says at that point rather than on a guess from day one.

## Using it day to day

You run everything by talking to your coding agent in your study folder. You never need to edit a file yourself.

### Creating a set

Ask for the set on the day you'll do it (or the night before):

> *"Make today's set."*
> *"Build the set for Tuesday."*
> *"Make me 20 timed SEC questions, mostly Hard."*

The agent checks your plan and error log, pulls matching questions from the College Board question bank, and saves a printable PDF and answer key to `sets/`.

### Uploading your results

When you finish a set or a practice test, give the results straight to the agent. You don't need to fill in any file or form:

> *"Set F answers: 1 B, 2 C, 3 A, …"*
> *"Here's my practice test score report."* (attach the screenshot or PDF)
> *"Practice test 2: RW 660, Math 670."*

The agent grades the set against its key and logs every miss in the error log. It asks you **why** you missed each one (`content`, `misread`, `careless` or `timing`), with a suggested answer for each. Then it updates the dashboard.

### Marking a day complete

> *"Mark today done."*
> *"Mark Saturday done."*

The day gets a check mark on the dashboard calendar, along with its score if it had a set.

### Anything else

> *"Add vocab: tenuous — weak, barely holding together."*
> *"What should I focus on this week?"*
> *"I can't study Thursday. Move it."*

## All questions come from the College Board

**Every question in every set comes from the official [SAT Suite Question Bank](https://satsuitequestionbank.collegeboard.org/).** The prompt doesn't allow the agent to write its own questions, passages or answer choices. AI-written questions can have wrong answer keys or test things the real SAT doesn't, and practicing on them can teach you the wrong habits.

The agent's job is to *choose* questions: by domain, skill and difficulty, and by what your error log says you need. When it explains a miss, it uses College Board's own rationale for that question.

## How it works

Each file in your study folder has one job, and the agent follows a fixed set of rules for each one:

- **The error log is the source of truth.** Every missed question is logged with its domain, the rule it broke, the letter you picked, the correct letter, and why you missed it. Before helping you, the agent reads the log, because it shows what you actually get wrong rather than what you think you get wrong.
- **The dashboard shows the data and never holds it.** All of your data goes in one `DATA` block at the top of `dashboard.html`. The agent edits that block and never touches the render code.
- **Sets are built from what you miss.** A set can re-test a weak rule on fresh questions instead of repeating ones you've already seen.
- **The projected score is honest about its uncertainty.** The dashboard shows a projected total with a 95% interval. The projection is anchored on your last full practice test and moves only on evidence logged after it. [`docs/projection.md`](docs/projection.md) explains every number.

## Rules the agent follows

Each of these was learned the hard way:

1. **Read the error log before giving advice.**
2. **Edit `DATA`, never the render code.**
3. **Every miss gets a `why` tag.** A pile of untagged misses can't tell you whether your gap is content or execution.
4. **Never write its own questions.** Every question comes from the College Board question bank.
5. **Don't pool a single domain into the projection until it has about 20 questions in one sitting.** With fewer questions, one miss looks like a 70-point drop.
6. **Confidence is not a progress metric between full tests.** Logging practice widens the interval, and only a full test narrows it again. That is intended behavior, not a bug.
7. **Never refetch the question bank.** Fetch it once and cache it locally.

## Your data stays on your computer

Your plan, error log, scores, vocabulary and practice sets are all created in your own folder. Nothing is sent back to this repo.

## Question content

This repo contains **no College Board questions**. The agent downloads them from the public question bank onto your own machine, for your own study. Don't share or republish them.

SAT is a trademark of the College Board, which is not affiliated with this project and does not endorse it.

## License

[MIT](LICENSE)
