# AI Code Review Checklist

**A 23-point pre-merge checklist for reviewing AI-generated code.** Catch
silent logic inversions, phantom APIs and swallowed errors before you merge.
Free, MIT licensed, no signup.

---

## Why this exists

AI-generated code contains roughly **1.7x more bugs** than human-written
code, and AI-authored pull requests carry about **75% more logic and
correctness errors** (CodeRabbit, across 470 GitHub repositories).

The gap shows up in review outcomes: **AI-generated PRs are accepted at
32.7%, against 84.4% for human-written ones.** And it is getting more
expensive — teams with high AI adoption merge 98% more pull requests, but
their review time has gone up 91%.

The reason is not that models are bad at code. It is that they were trained
on code humans had already published — code that had already passed review,
already looked right. They learned the *appearance* of correctness extremely
well and the substance of it only incidentally.

Which is why prompting better does not fix this. Code that looks correct and
is wrong is the failure mode, and the fix is a review process that assumes
the code looks fine and checks whether it is fine.

---

## The checklist

Run it before merging any code an AI assistant generated. About four minutes,
ordered by what costs you most when missed.

### STOP — never merge if any of these fail

- [ ] **No string-built queries.** Every query uses parameterized bindings.
      Search the diff for concatenation or f-strings inside a query call.
- [ ] **Authorization, not just authentication.** The code checks that *this
      user* may do *this action* to *this record* — not merely that someone
      is logged in. AI reliably writes the login check and skips the
      ownership check.
- [ ] **No secrets in the diff.** Including in tests, fixtures, comments and
      example config.
- [ ] **Every dependency it added actually exists.** Check each new import
      against the real registry. Models hallucinate plausible package names,
      and attackers register those names.
- [ ] **Nothing destructive runs unguarded.** Every delete, drop, truncate or
      bulk update is scoped, reversible, or gated. Confirm the WHERE clause
      exists and is correct.
- [ ] **No loosened security defaults.** No disabled TLS verification, no
      wildcard CORS, no auth middleware quietly removed to make a test pass.

### CORRECTNESS — it runs, but is the answer right?

- [ ] **Read every conditional out loud.** Flipped booleans, `>` vs `>=` and
      inverted guards are the most common AI logic bug. They never crash —
      they return wrong answers forever.
- [ ] **Check the boundaries.** Zero, one, empty, null, maximum.
- [ ] **Verify each API signature against real docs**, not against what the
      model said it was.
- [ ] **Confirm the error path.** Does each `catch` handle the error, or
      swallow it and continue with bad state?
- [ ] **Follow the money and the math.** Any arithmetic on currency, dates,
      percentages or units gets checked by hand with one real example.
- [ ] **Run it once with input you know the answer to.** Not the test suite —
      you, with a value you can verify.

### INTEGRATION — does it fit the codebase, or fight it?

- [ ] **It didn't reinvent something you already have.** Grep the repo for
      the concept, not the name. The AI cannot see your whole codebase, so it
      rebuilds what it cannot find.
- [ ] **It follows your existing patterns.** If it introduced a new one, that
      was the AI's architectural decision, not yours.
- [ ] **No new global state or hidden side effects.**
- [ ] **It scales past your test data.** Queries inside loops, full-table
      loads, missing pagination. Ten rows hides all of these.
- [ ] **Nothing unrelated changed.** Read the whole diff — AI edits often
      include drive-by reformatting and deleted comments.

### TESTS — do they prove anything?

- [ ] **The tests would fail if the code were wrong.** Break it on purpose
      and confirm the test goes red.
- [ ] **Tests cover the failure cases**, not just the happy path.
- [ ] **No mocked-away logic.** If the mock does the thing under test, the
      test is decorative.

### OWNERSHIP — the part everyone skips

- [ ] **You can explain every line** out loud, without the AI.
- [ ] **You know why it chose this approach, and you agree.**
- [ ] **You could rewrite it from scratch** if you had to.

---

## The 30-second version

If you only have time for three:

1. **Read every `if` out loud** — logic inversions are silent and permanent.
2. **Grep for what it just built** — it cannot see your codebase.
3. **Explain the diff without the AI** — if you can't, don't merge.

---

## Use it in your repo

Drop it into your pull request template:

```bash
curl -o .github/pull_request_template.md \
  https://raw.githubusercontent.com/YOUR-USERNAME/ai-code-review-checklist/main/CHECKLIST.md
```

---

## The eight failure modes

The bugs are not randomly distributed. They cluster into eight shapes:

| # | Mode | What it looks like |
|---|---|---|
| 1 | Silent Inversion | Flipped boolean or wrong operator. Never crashes, always wrong. |
| 2 | Phantom API | Methods and packages that don't exist — until an attacker registers the name. |
| 3 | Reinvented Wheel | Rebuilds a helper you already have. They disagree on rounding. |
| 4 | Swallowed Error | Caught, logged vaguely, execution continues with corrupted state. |
| 5 | Happy Path Only | You described success. It built success. |
| 6 | Confident Default | It made an architectural decision and you inherited it. |
| 7 | Ten-Row Illusion | Perfect on test data, catastrophic on real data. |
| 8 | Decorative Test | Green, covers the lines, incapable of failing. |

---

## Contributing

Found a failure mode that isn't here? Open an issue. Checks that have caught
real bugs in your codebase are especially welcome — this list should be
grounded in things that actually happened, not things that theoretically
could.

## Licence

MIT. Use it, fork it, put it in your team's PR template, print it out. No
attribution required.

---

<sub>This checklist is the free part of **The AI Code Review Kit**, which adds
the full failure atlas, 25 adversarial review prompts, a drop-in `REVIEW.md`
for Claude Code and Cursor, a context discipline protocol, and a git
pre-commit hook. [Details here](#) — but the checklist above is complete and
free forever, and you do not need the kit to use it.</sub>
