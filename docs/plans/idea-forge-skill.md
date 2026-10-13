# Plan: `/idea-forge`, a Claude Code skill that debates, improves, and maps your idea

- **Status:** planned (nothing built yet)
- **Date:** 2026-10-13
- **Reference:** Nate Herk's `/roast-room` ([post](https://x.com/nateherk/status/2109841753628914090), [repo](https://github.com/nateherkai/roast-room), MIT license)

## 1. What we're building

You type `/idea-forge <your idea>` in Claude Code. A council of seven sub-agents argues about the idea, builds better versions of it, and scores every version. You get back three things:

1. **A verdict:** GO, GO WITH CONDITIONS, RESHAPE, WAIT, or DON'T DO IT.
2. **The strongest version of the idea.** Often this is not the version you brought in.
3. **A path to success:** the cheapest test, 30/60/90-day milestones with numbers, the metrics to watch, kill criteria, and a date to revisit.

How the skill covers each part of the request:

| You asked for                      | How the skill does it                                                                                                                                                                                                                                |
| ---------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Debates your ideas                 | Seven seats each own one question and argue across three rounds. Round 1 runs in isolation so no seat anchors on another. A dedicated Contrarian seat attacks the idea. If every seat agrees at once, the Chair treats that as a warning, not a win. |
| Brainstorms ways to improve it     | Every seat proposes upgrades in Round 1. The Chair forges them into two or three concrete remixes, and you pick which ones go into the debate.                                                                                                       |
| Delivers the best path for success | The council ratifies a plan: the final shape of the idea, the cheapest test, milestones, metrics, mitigations from a pre-mortem, and kill criteria.                                                                                                  |

## 2. How `/roast-room` works (the reference)

Facts below come from the repo's `SKILL.md`, `references/`, and `.claude/agents/`.

- **The Chair is the main Claude Code thread.** It never argues a side. It runs the rounds, keeps score, and writes the verdict.
- **Seven seats are sub-agents** (`.claude/agents/council-*.md`, tools `Read, Grep, Glob`, `maxTurns: 12`). Each seat simulates a real person: Bezos, Paul Graham, Hormozi, Gary Vaynerchuk, Dan Martell, Gino Wickman, and Buffett. Each reads its own "brain" folder compiled from that person's public work. The Hormozi brain, for example, comes from 124 videos, has 8 files and about 870 lines, and gives every quote a source id.
- **Protocol:**
  1. Brief. If key facts are missing, ask at most four questions.
  2. Set a relevance weight of 1 to 3 for each seat.
  3. Round 1, opening positions, run in parallel. The Chair then notes the cruxes.
  4. Round 2, debate.
  5. The Chair drafts a resolution.
  6. Round 3: each seat endorses, amends, or dissents.
  7. A quote-check script runs, then the Chair writes `verdict.md` and a chat summary under 200 words.
- **Scoring:** a weighted average for each option, mapped to verdict bands. Three overrides can change the result: a ruin cap, a one-way-door cap, and a unanimity check. The report also shows spread and confidence.
- **Extras:** a local web "room" that replays the session files, PDF export, and three modes: `--quick` (4 seats, 2 rounds), `--seats a,b`, and `ask <seat>`.
- **Cost:** a full council is 21 sub-agent runs, about 1M tokens by the repo's own estimate.

## 3. What we keep, change, and drop

| Area         | roast-room                                                      | idea-forge                                                                                             | Why                                                                                                                                                                                                                                     |
| ------------ | --------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Seats        | Simulations of real people, built from compiled source "brains" | Archetype seats, each built around one lens, that cite public frameworks by name                       | Brains are the most expensive part to build, at hundreds of sources per seat. They also carry impersonation and quote-accuracy risk. Lens seats deliver most of the value for a fraction of the work. Real-person packs can come later. |
| Options      | Your options, plus "don't do it", plus a smaller version        | The same, plus a **Forge** step: seats propose upgrades, the Chair builds 2 to 3 remixes, and you pick | This covers the "brainstorm ways to improve it" ask.                                                                                                                                                                                    |
| Red team     | Implicit, through the unanimity check                           | A dedicated **Contrarian** seat plus a pre-mortem on the draft plan                                    | Sycophancy is the main way AI feedback fails.                                                                                                                                                                                           |
| Scoring math | The Chair does the arithmetic by hand                           | `score.mjs` computes everything from a JSON footer on each card                                        | No arithmetic mistakes. The repo already requires Node, so no Python is needed.                                                                                                                                                         |
| Verification | A Python script checks quotes against the brains                | Dropped. A card validator checks that each card has every field and valid ranges                       | No verbatim quotes of real people to verify.                                                                                                                                                                                            |
| Output       | Verdict report and PDF                                          | Verdict plus a **Path to success** section                                                             | This covers the "best path for success" ask.                                                                                                                                                                                            |
| Web room     | A local Node app replays the council                            | Not in v1                                                                                              | A good demo, but not core. Listed under Later.                                                                                                                                                                                          |

We reuse roast-room's protocol shape, verdict bands, overrides, and report structure under its MIT license, and credit it in `SKILL.md`. We do not copy its brains or seat files. The lens files are our own.

## 4. The council

| Seat           | Agent              | Owns the question                                                   | Frameworks it reasons with                                                                                                    |
| -------------- | ------------------ | ------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| **Strategist** | `forge-strategist` | Is this the right bet, and how reversible is it?                    | One-way and two-way doors, regret minimization, "what won't change", deciding at about 70% of the information (Bezos letters) |
| **Customer**   | `forge-customer`   | Does anyone want this badly enough to switch or pay?                | The Mom Test (Fitzpatrick), Jobs to Be Done (Christensen), "Do Things That Don't Scale" (Graham), retention curves            |
| **Money**      | `forge-money`      | Will people pay, how much, and do the unit economics work?          | Value Equation (Hormozi), LTV:CAC, payback period, value-based pricing                                                        |
| **Growth**     | `forge-growth`     | How will the first 100 customers hear about it?                     | Bullseye framework (Weinberg and Mares, _Traction_), underpriced attention, one channel first                                 |
| **Operator**   | `forge-operator`   | Who builds and runs this, and what does it cost the founder's week? | Theory of Constraints (Goldratt), time audit and buyback, accountability chart                                                |
| **Risk**       | `forge-risk`       | What is the worst case, and can you survive it?                     | Margin of safety, runway math, avoiding ruin, opportunity cost                                                                |
| **Contrarian** | `forge-contrarian` | What kills this, and what is the strongest case against it?         | Pre-mortem (Klein), inversion (Munger), base rates and the outside view (Kahneman), steelmanning the alternative              |

- **Weights:** before Round 1, the Chair gives each seat a weight of 1 to 3 for this idea and a one-line reason. Exactly one or two seats get a 3. The Contrarian is fixed at weight 1, so it cannot drag every score down by design. Its ruin flags still trigger the ruin cap at full force.
- **Natural tensions the Chair should expect:**
  - Strategist and Risk (long horizon, downside first) against Money and Growth (speed to cash, act now).
  - Customer (stay close to users, do things that don't scale) against Operator (systems and process).
  - The Contrarian against whichever option is leading.
- **Lens files** live at `lenses/<seat>.md`, about 80 to 120 lines each, modeled on roast-room's `_hot.md`. Each contains:
  - a worldview,
  - 6 to 8 decision rules,
  - 5 frameworks, each with how to apply it,
  - a decision loop,
  - signature questions,
  - voice notes (blunt, numbers first, no hedging),
  - "where this lens misfires".

## 5. The protocol (what `SKILL.md` tells the Chair to do)

```
Brief ──> Weights + session folder ──> R1: open + upgrades (7 parallel)
   ──> FORGE: Chair clusters upgrades into remixes ──> you pick (checkpoint)
   ──> R2: debate all options (7 parallel) ──> Chair drafts Path to Success
   ──> R3: ratify + Contrarian pre-mortem (7 parallel) ──> score.mjs ──> verdict
```

**Step 0, Brief.** Pull five things from the user's message:

1. the idea in one line,
2. who it is for,
3. current numbers (revenue, cash, team, and audience if relevant),
4. resources and limits (hours, budget, deadline),
5. what success looks like and by when.

Also record which way the user is leaning and why, because the council tests the lean instead of echoing it. If any of items 1 to 4 are missing, ask once with AskUserQuestion, at most four questions. If the user says "just run it", fill the gaps and mark each one `(assumed)`. Keep the brief under 300 words. Options:

- **A** is the idea as brought.
- **B, C, and so on** are alternatives the user named, plus the obvious smaller version.
- **Z** is always "don't do it" (keep doing what you do now).

**Step 1, Set up.** Set the weights. Create `forge-sessions/YYYY-MM-DD-<slug>/` and write `brief.md` and `brief.json` (options and weights).

**Step 2, Round 1: open and upgrade.** Spawn all seats in one message: parallel, foreground, `subagent_type: forge-<seat>`. Each card scores every option, flags ruin risk and the door type, names the single biggest weakness, and proposes **two upgrades**. An upgrade is a concrete change plus which score it moves, and by how much. Save the cards verbatim to `round-1.md`. Run `check-cards.mjs` and send any failing card back once. Add Chair notes:

- **Cruxes:** the 1 to 3 underlying disagreements, with the seats on each side.
- **Unanimity check:** if every seat backs the same option within 1 point, make the strongest unargued alternative a crux.

**Step 3, Forge.** This is the brainstorm. The Chair takes the 14 upgrades and builds **2 or 3 remixes**, and writes them to `remixes.md`. Rules:

- Each remix is the original plus a compatible bundle of upgrades, or a pivot several seats pointed at.
- Each remix is concrete: who it is for, what it is, the price, and the channel.
- Each remix names the seats whose upgrades it uses and what it trades away.
- The remixes are clearly distinct from each other.
- At least one remix is the smallest version that tests the riskiest assumption.
- The Chair adds nothing of its own. Every element traces back to a seat.

**User checkpoint:** show the remixes and ask with AskUserQuestion (multi-select) which ones enter the debate, plus "anything to add or change?". `--auto` skips the checkpoint. Cap the option set at 5: A, up to 3 remixes, and Z.

**Step 4, Round 2: debate.** All seats get the brief, `round-1.md`, `remixes.md`, and the cruxes. Each card:

- names the seat it disagrees with most, and gives a rebuttal,
- names what it concedes,
- answers each crux,
- scores every option,
- says whether it moved, and why,
- lists up to three checkable conditions for a yes.

The Contrarian's card attacks every option instead: the top two ways each one fails. Save the cards to `round-2.md`, then run `score.mjs` to produce interim scores.

**Step 5, Draft the Path to Success.** Write `draft-path.md`. It contains:

- the recommended option and verdict label,
- the final shape of the idea,
- merged conditions,
- the riskiest assumption and the cheapest test,
- milestones with dates,
- 2 or 3 leading metrics with targets,
- kill criteria.

Weigh arguments, not headcount: one seat with a decisive fact beats three seats with opinions. Apply the overrides from section 7.

**Step 6, Round 3: ratify.** Each seat votes Endorse, Endorse with amendment, or Dissent, and gives final scores. The Contrarian **pre-mortems the draft**: "It is 12 months from now and this failed. Here are the three most likely reasons, ranked, and a mitigation for each." The Chair adopts an amendment when no seat objects or when the argument for it went unanswered. Otherwise the amendment goes to the minority report. There is no fourth round.

**Step 7, Verdict.** Run `score.mjs` on the last round to write `scores.json`. Then write `verdict.md` from the template, and show a chat summary under 200 words:

- the label and the call,
- one line of option scores,
- the three arguments that carried the room (one line each, with the seat),
- the strongest objection,
- at most three next steps,
- the path to `verdict.md`,
- an offer: "ask any seat a follow-up, or rerun with new facts".

**Chair rules.**

- Never argue a side.
- Never rewrite a card.
- Raise a missed point by putting it to the closest seat as a crux, not by inserting it.
- State the mode and the number of runs used.
- Seats are lenses, not people. Nothing is attributed to a real person beyond naming a framework's author.

**Modes.**

| Mode                                    | Seats                                                                             | Rounds                                                                                                                       | Sub-agent runs |
| --------------------------------------- | --------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------- | -------------- |
| Full (default)                          | 7                                                                                 | 3                                                                                                                            | 21             |
| `--quick`                               | 4: the top two by weight, Risk or Strategist, and the Contrarian, who always sits | R1 and R2. R3 runs only if the top-option spread is more than 3 or a seat flags `ruin: likely`                               | 8 to 12        |
| `--seats a,b,c`                         | named seats                                                                       | 3                                                                                                                            | 3 × n          |
| `ask <seat> <question>`                 | 1                                                                                 | 1 consult, no scoring                                                                                                        | 1              |
| `--deep` (later)                        | 7                                                                                 | Adds a divergent round: each seat gets a forcing constraint (10x bigger, 10x cheaper, ship in 2 weeks, a different customer) | +7             |
| `revisit <session> <new facts>` (later) | same as the original session                                                      | Reruns from R2 with the prior session and the new numbers                                                                    | 14             |

## 6. Card format (Round 1 example)

Every card is plain text in the seat's voice and ends with a JSON footer that the scripts read:

````
SEAT: Customer (demand)
STANCE: Strong yes | Yes | Conditional yes | No | Strong no   (on the option you recommend)
OPTION SCORES: A n/10 · B n/10 · Z n/10
LENS SCORE: n/10   (your recommended option, judged on your lens alone)
CONFIDENCE: n/5
RUIN RISK: none | possible | likely
DOOR: one-way | two-way
THE CALL: one sentence naming an option or a variant.
WHY: 1-3 numbered arguments, each naming its framework.
BIGGEST WEAKNESS: the one thing most likely to sink option A.
UPGRADES:
1. <change> → moves <option/score> from n to n because ...
2. ...
WHAT WOULD CHANGE MY MIND: a concrete fact or number.
THIS WEEK: one action.
QUESTION FOR YOU: the one question to answer before committing.

```json
{"seat":"customer","round":1,"stance":"conditional_yes","scores":{"A":5,"B":7,"Z":3},"lens":6,"confidence":3,"ruin":"none","door":"two-way"}
```
````

The Round 2 and Round 3 cards follow roast-room's shape (rebuttal, concession, cruxes, moved?, conditions; then vote and amendment) and use the same JSON footer.

## 7. Scoring (implemented in `score.mjs`, documented in `references/scoring.md`)

- **Option score** = Σ(weight × seat score) / Σ(weight), to one decimal, taken from the last round that ran. The report shows the math line for each option.
- **Verdict bands** on the recommended option's score:

  | Score       | Verdict                      |
  | ----------- | ---------------------------- |
  | 7.5 or more | GO                           |
  | 6.0 to 7.4  | GO WITH CONDITIONS           |
  | 4.5 to 5.9  | RESHAPE                      |
  | below 4.5   | DON'T DO IT (also if Z wins) |

  **WAIT** replaces the band when the best option depends on an unmet trigger. WAIT always names the trigger, the date, and the fallback.

- **Overrides.** The script flags these; the Chair applies them and states them in the report.
  1. **Ruin cap:** if any seat ends at `ruin: likely` on the recommended option, cap the verdict at RESHAPE until that seat endorses a mitigation.
  2. **One-way-door cap:** if most seats call the option one-way and the spread is 4 or more, cap at GO WITH CONDITIONS and make the cheapest test a precondition.
  3. **Unanimity check (Round 1):** if every seat has the same top option within 1 point, the Chair forces a steelman of the strongest alternative in Round 2.
- **Spread** = highest seat score minus lowest seat score on the recommended option.
- **Confidence** = average seat confidence × 2, minus 1 if the spread is 5 or more.
- **Remix lift** = best remix score minus the original A's score. This measures whether the brainstorm earned its keep, and evals track it.
- **Support** = weighted average of the Round 3 final scores for the draft path.

## 8. Output: `verdict.md` template (abridged)

```markdown
# Idea Forge: <idea in a few words>

<date> · Seats: <list> · Rounds: <n> · Mode: <full | quick | named> · Runs: <n>

## Verdict: <LABEL> (<option>)

**The call:** <one or two sentences you can act on>

Scores: A x.x · B x.x · C x.x · Z x.x · Support x.x/10 · Confidence n/10 · Spread n · Remix lift +x.x
<override applied, if any>

## The idea, forged

<Final shape: what, for whom, price, channel. What changed from your original, and which seat drove each change.>

## Scorecard

<one row per seat: weight, stance, score per option, lens score, moved?, their one line>

## Why the council landed here

<3 to 5 bullets, each tagged with the seat and framework>

## Where they fought

<one block per crux: the question, who was on each side, who moved and why>

## Minority report

<the strongest dissent, and what would have to be true for it to be right>

## Path to success

- **Riskiest assumption:** ...
- **Cheapest test:** ... (at most 2 weeks and $<n>; the pass/fail number)
- **This week:** 1 to 3 actions
- **Day 30 / 60 / 90:** a checkable milestone for each
- **Metrics to watch:** 2 or 3 leading indicators, each with a target
- **Pre-mortem:** the top 3 ways this fails → a mitigation for each
- **Kill or pivot if:** ...
- **Revisit:** <date>, and bring <these numbers>

---

Simulated advisory lenses, not real people. This is not financial or legal advice.
```

## 9. File layout

```
.claude/
├── agents/
│   ├── forge-strategist.md      # frontmatter: name, description, tools: Read, Grep, Glob, maxTurns: 10
│   ├── forge-customer.md
│   ├── forge-money.md
│   ├── forge-growth.md
│   ├── forge-operator.md
│   ├── forge-risk.md
│   └── forge-contrarian.md
└── skills/
    └── idea-forge/
        ├── SKILL.md             # the Chair protocol (sections 5 to 8, condensed)
        ├── lenses/              # one file per seat (section 4)
        ├── references/
        │   ├── seats.md         # roster, weights, tensions, quick-mode picks
        │   ├── round-prompts.md # R1, R2, R3, consult, general-purpose fallback lines
        │   ├── scoring.md       # section 7
        │   └── report-template.md
        ├── scripts/
        │   ├── score.mjs        # cards → scores.json (no dependencies)
        │   ├── check-cards.mjs  # validates fields and ranges, exit 1 on a bad card
        │   ├── score.test.mjs   # node --test, using fixtures
        │   └── fixtures/
        └── evals/               # 6 briefs plus expectations.md (section 11)
forge-sessions/
├── examples/<one finished council>   # committed, so people can see the output
└── YYYY-MM-DD-<slug>/                # gitignored: ideas are private
```

`SKILL.md` frontmatter sketch:

```yaml
---
name: idea-forge
description: Use when the user wants an idea debated, stress-tested, improved, or turned into a plan. Triggers include "forge this", "debate my idea", "poke holes in this", "how do I make this better", "is this a good idea", "what's the best path", or when the user is undecided on a business, product, pricing, hiring, or career move. Seven lens sub-agents argue it over three rounds, forge improved versions, and the Chair returns a scored verdict and a path to success. Also handles one-seat consults ("ask the Contrarian").
argument-hint: "[your idea] [--quick] [--auto] [--seats a,b] | ask <seat> [question]"
---
```

**Implementation notes:**

- **Run in the main thread.** The skill must not use `context: fork`, because sub-agents cannot spawn sub-agents.
- **Seats are read-only.** Only the Chair writes session files.
- **Agent files load at session start.** In the session that creates them, spawn `general-purpose` with "Read `.claude/agents/forge-<seat>.md` and follow it as your instructions" first. roast-room uses the same fallback.
- **No-sub-agent runtimes** (claude.ai, or anywhere else that cannot spawn agents): the Chair runs the seats one after another and clears context between Round 1 seats, so earlier cards don't anchor later ones.
- **Cost lever:** seats can set `model: sonnet` in their frontmatter while the Chair stays on the session model. Decide this after the evals compare quality.
- **Lint:** `npm run lint` lints `.claude/**/*.mjs`, because ESLint 9 flat config does not skip dot-folders. Keep the scripts lint-clean so CI covers them too.

## 10. Build phases

Each phase ends with the checks in its "Done when" column.

| Phase            | Work                                                                                                                                                                                                                    | Done when                                                                                                            |
| ---------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| **0. Repo prep** | `CLAUDE.md`: add `.claude/skills/`, `.claude/agents/`, and `forge-sessions/` to "Allowed". `.gitignore`: add `!.claude/skills/`, `!.claude/agents/`, `forge-sessions/*`, and `!forge-sessions/examples/`.               | `git check-ignore` confirms skill and agent files are tracked and session folders are ignored.                       |
| **1. Seats**     | 7 lens files and 7 agent files.                                                                                                                                                                                         | `ask <seat>` returns an answer clearly in that lens. On the same brief, at least 3 seats disagree on the top option. |
| **2. Chair**     | `SKILL.md` and the 4 reference files. Run one full council end to end on a real idea.                                                                                                                                   | A full session folder exists with every file from section 5. The chat summary is under 200 words.                    |
| **3. Scripts**   | `score.mjs`, `check-cards.mjs`, and tests with fixtures. Wire them into Steps 2, 4, and 7.                                                                                                                              | `node --test .claude/skills/idea-forge/scripts/` passes. The numbers in `verdict.md` match `scores.json`.            |
| **4. Modes**     | `--quick`, `--seats`, `ask`, and `--auto`.                                                                                                                                                                              | Each mode runs once and uses the expected number of sub-agent runs.                                                  |
| **5. Evals**     | 6 briefs and expectations (section 11). Tune the lens files and prompts until they pass. Commit one example session.                                                                                                    | At least 5 of 6 expectations met. The card validator passes on 100% of cards.                                        |
| **Later**        | `--deep` and `revisit`; real-person persona packs (roast-room's brain format plus a quote checker); PDF export; a dev-only session viewer page in this Next.js app; packaging as a plugin so it works in every project. | Each one gets its own plan when picked up.                                                                           |

## 11. Evals (`evals/`)

| #   | Brief                                                                                   | Expectation                                                                              |
| --- | --------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| 1   | **Hidden fatal flaw:** a tool that depends on scraping a platform whose terms forbid it | At least one seat flags `ruin: possible` or `likely`. Option A ends at RESHAPE or lower. |
| 2   | **Strong wrong lean:** the user wants to quit their job with 2 months of runway         | The lean is challenged by name. The one-way-door cap or ruin cap fires.                  |
| 3   | **Genuinely good idea:** small, reversible, with demand already validated               | GO or GO WITH CONDITIONS. This catches the council being negative by reflex.             |
| 4   | **Vague idea:** one line, no numbers                                                    | The Chair asks at most 4 questions. With "just run it", every gap is marked `(assumed)`. |
| 5   | **Improvable idea:** decent, but aimed at the wrong customer segment                    | A remix outscores A (remix lift > 0), and the remix changes the segment.                 |
| 6   | **Stability:** rerun #3 and #5                                                          | Same verdict band and same top option as the first run.                                  |

Track the cost of each run (sub-agent runs, and wall time if visible) next to the results. The `skill-creator` skill can help run and compare these evals.

## 12. Risks

| Risk                                                        | Mitigation                                                                                                                    |
| ----------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| Archetype seats sound alike                                 | Sharp lens files with distinct decision rules and the "natural tensions" list. Phase 1 checks that seats disagree.            |
| Cost: 21 runs per full council                              | `--quick` for reversible calls; seats can move to a smaller model after the evals; the mode and run count go in every report. |
| Contrarian bias (everything gets RESHAPE)                   | The Contrarian is fixed at weight 1, and eval #3 catches it.                                                                  |
| Cards drift from the format                                 | The JSON footer plus `check-cards.mjs`, with one retry for each failing card.                                                 |
| The Chair sneaks in its own opinions while building remixes | The rule that every remix element cites a seat, plus the user checkpoint.                                                     |
| Agents are not registered in the session that created them  | The `general-purpose` fallback (section 9).                                                                                   |

## 13. Decisions made (change any of these before building)

1. **Name `/idea-forge`.** `/goal` is already a built-in Claude Code command (it sets a condition Claude checks before stopping), so a skill named `goal` would collide with it.
2. **Archetype lens seats, not simulations of real people**, for v1. Persona packs are listed under Later.
3. **It lives in this repo** under `.claude/`. That needs the Phase 0 changes to `CLAUDE.md` and `.gitignore`. To use it in every project, package it as a plugin later.
4. **Sessions stay out of git**, because ideas are private. One example session is committed.
5. **No web viewer in v1.**
6. **Relation to your existing `idea-vetting` skill:** `idea-vetting` ("vetting mode") stays as the fast, single-pass gut check. `idea-forge` is the full council. The Contrarian lens can borrow its interrogation questions.

## 14. Definition of done (ready for `/goal`)

To have Claude build this in one push, start a session on this repo and paste:

```
/goal /idea-forge is built per docs/plans/idea-forge-skill.md: phases 0-5 are complete; .claude/skills/idea-forge/ holds SKILL.md, 7 lens files, 4 reference files, score.mjs, check-cards.mjs and passing node --test tests; .claude/agents/ holds the 7 forge-* agents; all 6 eval briefs have a finished session with verdict.md and scores.json and at least 5 of 6 meet their expectation; one example session is committed under forge-sessions/examples/; npm run lint and npm run build pass; the .claude context files are updated; and the work is pushed with a PR open against main.
```
