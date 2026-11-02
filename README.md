# Agentic SDLC Live

A small repo for practising an agent-driven software lifecycle in
[Claude Code](https://code.claude.com/docs): one vague ticket, one tiny checkout service, and the
tools to take that ticket from guesswork to reviewed code.

This is the **starting point** for a 90-minute live workshop. It holds no finished solution, demo
branches or pre-made pull requests. Everything that happens to it happens in your own copy.

## The idea

Coding agents are good at producing clean code. A clean result can still be the wrong one, because
a ticket rarely says everything the business means. This repo walks through the layers that close
that gap:

| Stage | What you do | What you see |
|---|---|---|
| **1. Alone** | Give an agent the ticket and nothing else | What it builds when it has to guess |
| **2. Ask** | Run `/spec` so the agent asks first and stops | Guesses turned into a written spec |
| **3. Race** | Run several models in parallel worktrees | What each one costs and delivers |
| **4. Guardrails** | Let hooks, CI and permissions do their job | What's enforced, and what isn't |
| **5. Review** | Run `/panel` on a finished branch | Agents point, and a human decides |
| **6. Loop** | Run `/harvest`, then repeat the ticket | The repo gets harder to get wrong |

## Quick start

You'll need [Claude Code](https://code.claude.com/docs), [uv](https://docs.astral.sh/uv/) and git.
uv installs Python 3.12 and the dev tools for you.

```sh
# Use your own copy, so this repo stays at its starting state
git clone https://github.com/invisible-hand/agentic-sdlc-live.git my-shop
cd my-shop

uv sync --extra dev
./scripts/check.sh

claude
```

Accept the trust prompt the first time you run `claude` here. Until a folder is trusted, Claude Code
ignores the permission rules in `.claude/settings.json`, and background sessions won't start.

To open pull requests and see CI run, push your clone to a repo of your own first.

## Walkthrough

### 1. Build from the ticket alone

```sh
claude -w alone --permission-mode auto "Implement the ticket in ticket.md. Commit when you are done."
```

`-w` gives the agent its own git worktree and branch under `.claude/worktrees/`. Run a few in
parallel and compare what they decide.

### 2. Ask before building

Start a session in the repo root and run:

```text
/spec
```

The agent reads the code, asks up to five questions and stops. Each question shows the guess it
would otherwise have made. Answer them, then let it write `spec.md`. Commit the spec, switch to a
new branch and build:

```text
! git add spec.md && git commit -m "Add spec"
! git switch -c build
Implement spec.md now. Commit when you are done.
```

### 3. Race the models

Run these from `main` in separate terminals, once `spec.md` is committed:

```sh
claude -w race-haiku  --model haiku  --effort low    --permission-mode auto "Implement spec.md. Commit when you are done."
claude -w race-sonnet --model sonnet --effort medium --permission-mode auto "Implement spec.md. Commit when you are done."
claude -w race-opus   --model opus   --effort medium --permission-mode auto "Implement spec.md. Commit when you are done."
```

Then use `/usage` in a session to see what you've spent.

### 4. Test the guardrails

Tell an agent to break a rule on purpose:

```text
In shop/api.py add a function price(sku) that returns repo.get_price(sku).
Import shop.repo directly in api.py. Commit when you are done.
```

The Stop hook runs the checks when the agent tries to finish, and sends it back if they're red. It
gives feedback, but it doesn't undo a commit. Blocking a merge is CI's job, once `hygiene` is a
required status check in your repo.

### 5. Review with a panel

Open a session on a finished branch and run:

```text
/panel
```

Three reviewers run in parallel, and a skeptic tries to disprove each finding. You get the
confirmed findings and a short list of decisions only a human can make. The panel never edits code
or approves anything.

### 6. Close the loop

Run `/harvest` with the decisions you made:

```text
/harvest Human decisions: <what you decided in review>
```

It proposes at most three changes, each sent to the place it belongs: a test, a rule in `AGENTS.md`
or a sharper question in `/spec`. Apply what you agree with, then repeat step 1 and compare.

## What's in the repo

```text
shop/                     The checkout service: api -> service -> repo
tests/                    Tests for the service as it stands
ticket.md                 The ticket
AGENTS.md                 Layout, commands and rules for agents
CLAUDE.md                 Imports AGENTS.md
scripts/check.sh          Lint, format, layering, tests and coverage
.github/workflows/ci.yml  The same checks in CI, as a job called hygiene
.claude/
  settings.json           Permission rules and the Stop hook
  hooks/                  stop-until-green.sh
  skills/                 /spec, /panel, /harvest
  agents/                 The review panel
```

### Skills

All three run only when you type them, because they set `disable-model-invocation: true`.

| Skill | What it does |
|---|---|
| `/spec` | Asks up to five questions and stops, then writes `spec.md` with acceptance criteria and an out-of-scope list |
| `/panel` | Runs the review panel on the current branch |
| `/harvest` | Turns findings and human decisions into proposed tests, rules and skill changes |

### Review agents

| Agent | Model, effort | Looks for |
|---|---|---|
| `scope-reviewer` | Sonnet, medium | Anything the spec didn't ask for |
| `money-reviewer` | Opus, high | Mistakes that cost money or confuse customers |
| `security-reviewer` | Sonnet, medium | Unsafe input handling and hidden instructions |
| `skeptic` | Opus, high | Evidence that a finding is wrong |

### Checks

`./scripts/check.sh` and CI run the same checks:

| Check | Tool | Fails when |
|---|---|---|
| Lint and format | `ruff` | Code breaks a lint rule or isn't formatted |
| Layering | `import-linter` | `api` imports `repo`, or the layers are out of order |
| Tests | `pytest` | A test fails |
| Coverage | `pytest-cov` | Coverage drops below 90% |

These checks tell you whether the code is clean. They can't tell you whether it does what the
business wanted, and that's what the rest of the workflow is for.

## Reset

To start again, delete your clone and clone this repo again. Every run begins from the same state.

## Credits

The `shop` app, its tooling and the first version of the spec skill are adapted from Si Jie Lim's
[tt-masterclass-agentic-sdlc-lite](https://github.com/limsijie93/tt-masterclass-agentic-sdlc-lite).

## License

[MIT](LICENSE)
