# Ethan Simpson

I build and operate **TheOnlyCopy**, a small SaaS that sends personalized intelligence briefs to business teams. It is a solo operation: one person and one production system, run largely through coding agents with mechanical checks around them.

This profile exists to show *how* that system is built and operated, not to sell it.

## What I work on

- **A relevance pipeline that ranks news for a specific reader.** In one measured week in September 2026, about 158,500 article-profile pairs got a relevance score from a language model. Every newly ingested article is paired with every subscriber profile. A cheap keyword prefilter zeroes out obvious misses, and the model scores the rest. For each team member who has set their own preferences, the scored pool is re-ranked against those interests and filled through a tiered waterfall, so two people at the same company can get different briefs.
- **An agentic operating model with a boundary.** Three actors share the work. The advisor, me plus a planning model, makes the calls. CC, a coding agent, owns the source and also makes some production changes, such as applying migrations. Claw, an operations agent on a separate host, handles monitoring, alerting and much of live operations, and reads the source through a mirror. Larger build plans are written up before code lands. Some get a written pre-audit by Claw and some get an adversarial multi-agent review with my sign-off. The pre-audit channel carries far more asks than answers.
- **Incidents that become mechanisms.** The rule is one incident, one structural lesson, and wherever possible a test or deploy-time check that would have caught it. There are 107 numbered structural lessons so far, and 19 rules in the pre-deploy check.

The architecture, the pipeline as a ranking problem, and four incident write-ups live in **[toc-architecture](https://github.com/ethanjsimp-debug/toc-architecture)**.

## The system by the numbers

Measured from the repository, its CI and the production database on 2026-09-23 and 2026-09-24. Values marked with a tilde are rounded. The weekly figures are one measured week, except the median in the pairing row, which spans the four weeks before it.

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
