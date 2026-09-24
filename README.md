# Ethan Simpson

I build and run software systems solo, mostly through coding agents, with mechanical checks around them. Two are in production today, on different stacks and for different users: TheOnlyCopy, a SaaS that sends personalized intelligence briefs to business teams, and an advisory chat bot for a client in a regulated industry. A third, a gated preview site for a homebuilder, is deployed. Two more are built and tested but not running. Each block below says which is which.

This profile exists to show *how* these systems are built and operated, not to sell them.

## What I run

### In production

**TheOnlyCopy.** Personalized intelligence briefs for business teams, on Next.js, Supabase and Anthropic models on Vercel, with an operations agent on a separate host.

- A relevance pipeline that ranks news for a specific reader. In one measured week in September 2026, about 158,500 article-profile pairs got a relevance score from a language model, after a cheap keyword prefilter zeroed out obvious misses. For each team member who has set their own preferences, the scored pool is re-ranked and filled through a tiered waterfall, so two people at the same company can get different briefs.
- A three-actor operating model. The advisor, me plus a planning model, makes the calls. CC, a coding agent, owns the source. Claw, an operations agent on a separate host, handles monitoring, alerting and much of live operations, and reads the source through a mirror.
- The rule is one incident, one structural lesson. There are 107 numbered structural lessons and 19 rules in the pre-deploy check.
- The architecture, the pipeline as a ranking problem, and four incident write-ups live in **[toc-architecture](https://github.com/ethanjsimp-debug/toc-architecture)**. Its numbers are in the table below.

**An advisory bot for a client in a regulated industry.** A Slack bot that answers a team's questions about an external automation platform. TypeScript on Node 22, running as a systemd service on a private host, with SQLite storage and Anthropic models.

- Advisory-only by construction. A test parses the platform client's source and fails on any write verb or non-GET request. Workflow examples come from a local corpus and are never fetched from the live platform, after the context-window blowout described in the incident record. The only live call left is a read-only listing of credential names and types.
- Every model call passes a pre-send token guard. A proposed workflow goes through a deterministic validator, gets at most two self-repair rounds and one escalation to a larger model, and is withheld if it still fails.
- The test suite is split by what it reads. 98 hermetic tests run in CI on every push and pull request to the main branch. The 12 tests that need the live knowledge corpus run inside the deploy script on the host, and the service's start path refuses any build that script did not verify.
- 13 commits since 2026-06-27.

### Deployed preview

**A preview site for a homebuilder.** A gated preview of three design directions for a homebuilder's website, with an interactive 3D walkthrough of one floor plan. Next.js 16, React 19 and three.js on Vercel.

- Every design theme renders the same typed content module. Per-theme parity tests check that each theme implements every component in the contract, uses the shared legal and lead-form components, and does not hardcode the phone number, the address, the listed square footages or the efficiency figure.
- The walkthrough merges meshes by material and shadow flags and draws a frame only when something changes. By the project's own measurements, draw calls in the heaviest view fell from 1,139 to 86, and sit at 91 after a later realism pass, against a budget of 150. The standalone build refuses to run when the baked lighting file no longer matches the plan geometry. The deployed site checks the same fingerprint at runtime and falls back to unbaked lighting with a warning.
- Each form sink returns a retained flag. The log-only sink, and any misconfiguration that falls back to it, always reports false, so the preview cannot claim it captured feedback that it only logged. For the email sink the flag means the provider accepted the message, not that it was delivered.
- Deployed behind an access-code gate and closed to search indexing. 31 commits on two days between 2026-08-27 and 2026-08-29, 80 test cases, no CI.

### Built, not running

**A measurement ledger for billing on verified savings.** A TypeScript monorepo with an append-only Postgres ledger and a deterministic engine for the billing path. So far the engine computes a baseline value and an opportunity score. The certification step that would turn measured savings into an invoiceable figure is deferred.

- The measurement tables are append-only in the database itself. Triggers on them reject UPDATE, DELETE and TRUNCATE, and on every ledger table the application role is granted only SELECT and INSERT.
- Money is integer minor units only. Division is allowed in one function, with banker's rounding. Two independent scanners over the engine's source, one using the TypeScript tokenizer and one working byte by byte, fail if a division, modulo or right-shift operator appears anywhere else.
- A determinism gate runs the engine twice, in different time zones and locales and with the input order shuffled, and byte-compares the output.
- CI runs formatting, type checks, the build and the full suite on every push and pull request. In the latest run on main, 846 tests passed and 11 live-tenant tests were skipped. It is not deployed. Its mail and calendar pulls are blocked in code, because both window ceilings ship unset and a person has to set them. Its model adapter has never made a live call. 203 commits between 2026-08-17 and 2026-08-31.

**A paper-trading research system.** A Python research, backtesting and paper-trading harness for same-day options, run largely by coding agents in unattended mode.

- The broker adapter the automated session uses is hard-coded to the paper endpoint. It asserts the paper URL when it is built and again before every order it sends. An older execution path picks paper or live from a config flag, and that flag is set to paper.
- A fix was once committed while a session was already running an older checkout. The launcher now aborts unless every required fix commit is an ancestor of the checked-out HEAD.
- A hypothesis engine reports Bonferroni and Benjamini-Hochberg corrections across every hypothesis evaluated in a campaign. Within one engine instance, its holdout slice is a one-shot latch that raises an error on a second read.
- 1,756 test functions in 196 files, and no CI. Paper trading only, and not running now. 520 commits between 2026-05-22 and 2026-06-27.

## TheOnlyCopy by the numbers

These figures are TheOnlyCopy's alone, not the portfolio's. They were measured from its repository, its CI and its production database on 2026-09-23 and 2026-09-24. Values marked with a tilde are rounded. The weekly figures are one measured week, except the median in the pairing row, which spans the four weeks before it.

| Measure | Value |
|---|---|
| Commits on the main branch | 2,444, the first on 2026-03-28 (commit c0f403a9) |
| Tests passing in the latest green CI run | 9,747 across 762 test files |
| Article-profile pairs scored by a model, per week | ~158,500, about 9,300 per scored profile |
| Articles paired with each profile, per week | ~12,650 in the measured week (median 12,756 across 53 earlier customer profile-weeks) |
| Articles clearing relevance 70 for the median customer, per week | ~210 |
| TypeScript in the application source, excluding tests | ~223,000 lines |
| API route files | 238 |
| Cron schedule entries | 100, covering 99 distinct jobs |
| SQL migration files, schema and data | 173 |
| Numbered structural lessons | 107 |
| Pre-deploy mechanical rules | 19 |

Stack: Next.js 14 on Vercel, Supabase (Postgres), Stripe, Resend, and Anthropic models. Sonnet writes the briefs and tags articles. Haiku scores relevance and runs the fact and integrity checks.

## How I work with agents

- **Code and data are different substrates.** CC is trusted for claims about what the code does. Claw is trusted for claims about what the rows say. A claim that joins the two, such as "this code caused that row," needs confirmation from both sides before anyone acts on it.
- **Bytes travel, references don't.** Content one agent needs from another goes into the message itself or into the shared repository. "See the file in my workspace" resolves to nothing across the boundary.
- **Observation before enforcement, for the riskiest changes.** Risky automated behaviors ship behind a flag and first run in an observe mode that logs what they would have done, then get switched on one flag at a time.
- **A permanently red gate is a cut alarm wire.** The rule is that a check which can't be kept green gets fixed or removed, never normalized.

## Contact

Ethan Simpson, a senior at Brown University studying Applied Mathematics-Economics. Email: [ethanjsimp@gmail.com](mailto:ethanjsimp@gmail.com)
