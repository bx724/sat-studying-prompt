# SAT Command Center

A study system for the digital SAT that runs on an AI coding agent such as [Claude Code](https://claude.com/claude-code). You give it your test date, your baseline score and your target. It sets up a study folder, keeps an error log of every question you miss, builds practice sets from the official College Board question bank, and keeps a dashboard up to date with a projected score.

> **Status: work in progress.** The prompt, the dashboard template and the bank tools are being pulled out of a real month of SAT prep and cleaned up for general use.

## How it works

Each file in the study folder has one job, and the agent follows a fixed set of rules for each one:

- **The error log is the source of truth.** Every missed question is logged with its domain, the rule it broke, the letter you picked, the correct letter, and why you missed it (`content`, `misread`, `careless` or `timing`). Before helping you, the agent reads the log, because it shows what you actually get wrong rather than what you think you get wrong.
- **The dashboard shows the data and never holds it.** `dashboard.html` is a fixed template. All of your data goes in one `DATA` block at the top of the file. The agent edits that block and never touches the render code.
- **Sets are built from what you miss.** The bank tools pull questions by domain, skill and difficulty, so a set can re-test a weak rule on fresh questions instead of repeating ones you've already seen.
- **The projected score is honest about its uncertainty.** The header shows a projected total with a 95% interval. The projection is anchored on your last full practice test and moves only on evidence logged after it.

## Rules the agent follows

Each of these was learned the hard way:

1. **Read the error log before giving advice.**
2. **Edit `DATA`, never the render code.**
3. **Every miss gets a `why` tag.** A pile of untagged misses can't tell you whether your gap is content or execution.
4. **Don't pool a single domain into the projection until it has about 20 questions in one sitting.** With fewer questions, one miss looks like a 70-point drop.
5. **Confidence is not a progress metric between full tests.** Logging practice widens the interval, and only a full test narrows it again. That is intended behavior, not a bug.
6. **Never refetch the bank.** Fetch it once and cache it locally.

## Repository layout (planned)

```
sat-studying-prompt/
├── PROMPT.md              # the prompt that bootstraps and runs a study folder
├── dashboard.html         # dashboard template with an empty DATA block
├── examples/demo-data.js  # made-up student data, to see the dashboard working
├── bank/                  # fetch / parse / select / render tools
├── docs/projection.md     # how the projected score and confidence are computed
└── workspace/             # YOUR data goes here (git-ignored)
```

## Quick start (planned)

1. Clone this repo.
2. Open it in Claude Code (or another coding agent) and point it at `PROMPT.md`.
3. Answer the setup questions: test date, baseline section scores, target score and weekly hours.
4. Open `workspace/dashboard.html` in a browser.

## Your data stays local

Everything personal (your plan, error log, scores, vocabulary, generated sets and the cached question bank) lives in `workspace/`, `bank/cache/` and `sets/`, and all three are git-ignored. If you contribute back, nothing personal goes with it.

## Question content

This repo contains **no College Board questions**. The bank tools fetch questions from the public [SAT Suite Question Bank](https://satsuitequestionbank.collegeboard.org/) onto your own machine, for your own study. Don't commit the fetched content.

SAT is a trademark of the College Board, which is not affiliated with this project and does not endorse it.

## License

TBD.
