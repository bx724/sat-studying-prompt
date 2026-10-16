# SAT Command Center

This repository contains the prompt and instructions to allow you to create your own tailored SAT studying dashboard. Please read this README before using the prompt.

The prompt turns an AI coding agent such as [Claude Code](https://claude.com/claude-code) or [Codex](https://openai.com/codex/) into a study system for the digital SAT. It plans your schedule, keeps an error log of every question you miss, builds practice sets from the official College Board question bank, and keeps a dashboard up to date with a projected score.

> **Status: work in progress.** The prompt is being pulled out of a real month of SAT prep and cleaned up for general use.

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

## How it works

Each file in your study folder has one job, and the agent follows a fixed set of rules for each one:

- **The error log is the source of truth.** Every missed question is logged with its domain, the rule it broke, the letter you picked, the correct letter, and why you missed it (`content`, `misread`, `careless` or `timing`). Before helping you, the agent reads the log, because it shows what you actually get wrong rather than what you think you get wrong.
- **The dashboard shows the data and never holds it.** All of your data goes in one `DATA` block at the top of `dashboard.html`. The agent edits that block and never touches the render code.
- **Sets are built from what you miss.** The agent pulls questions by domain, skill and difficulty, so a set can re-test a weak rule on fresh questions instead of repeating ones you've already seen.
- **The projected score is honest about its uncertainty.** The dashboard shows a projected total with a 95% interval. The projection is anchored on your last full practice test and moves only on evidence logged after it.

## Rules the agent follows

Each of these was learned the hard way:

1. **Read the error log before giving advice.**
2. **Edit `DATA`, never the render code.**
3. **Every miss gets a `why` tag.** A pile of untagged misses can't tell you whether your gap is content or execution.
4. **Don't pool a single domain into the projection until it has about 20 questions in one sitting.** With fewer questions, one miss looks like a 70-point drop.
5. **Confidence is not a progress metric between full tests.** Logging practice widens the interval, and only a full test narrows it again. That is intended behavior, not a bug.
6. **Never refetch the question bank.** Fetch it once and cache it locally.

## Your data stays on your computer

Your plan, error log, scores, vocabulary and practice sets are all created in your own folder. Nothing is sent back to this repo.

## Question content

This repo contains **no College Board questions**. The agent fetches questions from the public [SAT Suite Question Bank](https://satsuitequestionbank.collegeboard.org/) onto your own machine, for your own study. Don't share or republish the fetched content.

SAT is a trademark of the College Board, which is not affiliated with this project and does not endorse it.

## License

TBD.
