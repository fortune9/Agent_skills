# Running the zz-vibe-coding discovery evals

Goal: verify that `zz-vibe-coding` is invoked for the prompts in
`discovery-cases.md` that expect `trigger`, and NOT invoked for those that
expect `no-trigger`.

Skill discovery depends only on the **initial prompt** in a fresh context, so
each case must run in its own fresh agent. Do not run them all in one
conversation — earlier cases would pollute the context of later ones.

## Method A — dispatch subagents (in Claude Code)

Ask your top-level agent (the one you're chatting with) to run the harness:

> Run the zz-vibe-coding discovery evals in
> `~/.claude/skills/zz-vibe-coding/evals/discovery-cases.md`.

For **each** case, the orchestrator dispatches ONE subagent whose entire
prompt is the case's fenced prompt text, verbatim, prefixed with the harness
instruction below. It then inspects the subagent's transcript for a
`zz-vibe-coding` skill invocation.

Harness instruction prepended to every case prompt:

```
You are being given a realistic user request. Respond exactly as you normally
would at the START of handling it: first invoke any skills that apply, then
give your opening response. Do NOT write the full solution — stop after your
first substantive step so we can observe which skills you chose. Announce any
skill you invoke as "Using <skill> ...".

--- USER REQUEST ---
<CASE PROMPT GOES HERE>
```

Scoring per case:
- Subagent invoked `zz-vibe-coding` → observed = `trigger`
- Subagent did not → observed = `no-trigger`
- PASS if `observed == expect`, else FAIL.

Emit a results table: `id | expect | observed | pass?` and a summary count.

### Important harness notes

- **Fresh context per case is mandatory.** One subagent = one case.
- The subagent must have the same skills available as your real setup
  (it does, when dispatched normally in Claude Code). If you sandbox the
  subagent so it can't see skills, the eval is meaningless.
- Detect invocation from the actual tool call / announcement, not from the
  prose. A subagent may *mention* vibe coding without invoking the skill —
  that is `no-trigger`.
- Run 3 reps per case if you want a stability signal; discovery can be
  probabilistic near the boundary (esp. N4, N5). Report majority + note
  variance, per the writing-skills "variance is a metric" guidance.

## Method B — manual spot check

For a quick check without a harness: open a fresh Claude Code session, paste a
single case's prompt, and observe whether it announces `Using zz-vibe-coding`.
Repeat with `/clear` between cases. Slower, but zero setup.

## Interpreting failures

- **A positive case (P*) does not trigger** → the `description` is too narrow
  or the prompt's coding intent is implicit. Widen the description's trigger
  wording or accept it as a known gap.
- **A negative case (N*) triggers** → the skill is over-eager. The description
  ("when coding is needed") is broad; if N1/N3/N6 trip, tighten it. N4/N5 are
  labelled borderline — a trigger there is a boundary judgment, not a hard bug.
- Treat this like TDD: a failing case tells you what to change in the
  `description`, then re-run. See `superpowers:writing-skills`.
```
