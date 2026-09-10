# The Pre-Merge Checklist for AI-Written Code

You didn't write it. You still own it.

Run this before merging any code an AI assistant generated. It takes about
four minutes and is ordered by what actually costs you the most when missed.

**Why this exists:** AI-generated code contains roughly 1.7x more bugs than
human-written code, and AI-authored pull requests carry about 75% more logic
and correctness errors. Not because the models are bad — because they write
code that *looks* right. Looking right is the failure mode. Reading it like
you'd read a stranger's PR is the fix.

---

## STOP — never merge if any of these fail

- [ ] **No string-built queries.** Every SQL/NoSQL query uses parameterized
      bindings. Search the diff for string concatenation or f-strings inside
      a query call.
- [ ] **Authorization, not just authentication.** The code checks *this user
      may do this action to this record* — not merely that someone is logged
      in. AI reliably writes the login check and skips the ownership check.
- [ ] **No secrets in the diff.** No API keys, tokens, connection strings, or
      passwords, including in tests, fixtures, comments, or example config.
- [ ] **Every dependency it added actually exists and is the real one.**
      Check each new import against the real package registry. Models
      hallucinate plausible package names, and attackers register those names.
- [ ] **Nothing destructive runs unguarded.** Any delete, drop, truncate,
      overwrite, or bulk update is scoped, reversible, or gated. Check the
      WHERE clause exists and is correct.
- [ ] **No loosened security defaults.** No disabled TLS verification, no
      wildcard CORS, no `permitAll`, no auth middleware quietly removed to
      "make the test pass."

## CORRECTNESS — the code runs, but does it compute the right answer?

- [ ] **Read every conditional out loud.** Flipped booleans, `>` vs `>=`, and
      inverted guard clauses are the single most common AI logic bug. They
      never crash — they just return wrong answers forever.
- [ ] **Check the boundaries.** What happens at zero, one, empty, null, and
      the maximum? AI writes the happy path by default.
- [ ] **Verify each API signature against real docs.** Not against what the
      model said. Confirm argument order, return type, and whether it throws
      or returns an error.
- [ ] **Confirm the error path.** Find every `catch` / `except`. Does it
      handle the error, or swallow it and continue with bad state? A bare
      catch that logs and proceeds is worse than a crash.
- [ ] **Follow the money and the math.** Any arithmetic on money, dates,
      percentages, or units gets manually checked with one real example.
      Floating-point currency and naive timezone handling are classic.
- [ ] **Run it once with input you know the answer to.** Not the test suite —
      you, with a value you can verify by hand.

## INTEGRATION — does it fit the codebase, or fight it?

- [ ] **It didn't reinvent something you already have.** Search the repo for
      the function it just wrote. The AI can't see your whole codebase, so it
      rebuilds helpers that already exist. This is how duplication compounds.
- [ ] **It follows your existing patterns.** Same error handling style, same
      naming, same layering as the code around it. If it introduced a new
      pattern, that was the AI's architectural decision, not yours.
- [ ] **No new global state or hidden side effects.** Check whether it mutates
      anything outside its own scope.
- [ ] **It scales past your test data.** Look for queries inside loops (N+1),
      loading a full table into memory, missing pagination, and unbounded
      recursion. Ten rows hides all of these; production does not.
- [ ] **Nothing unrelated changed.** Read the *whole* diff. AI edits often
      include drive-by reformatting, deleted comments, or "improvements" to
      code you never asked it to touch.

## TESTS — do they actually prove anything?

- [ ] **The tests would fail if the code were wrong.** Break the function on
      purpose and confirm the test goes red. AI-written tests frequently
      assert that the code does what it does, which proves nothing.
- [ ] **Tests cover the failure cases, not just success.** At least one test
      per error path.
- [ ] **No mocked-away logic.** If the mock is doing the thing under test,
      the test is decorative.

## OWNERSHIP — the part everyone skips

- [ ] **You can explain what every line does.** Out loud, without the AI. Any
      line you can't explain is a line you cannot debug at 2am.
- [ ] **You know why it chose this approach, and you agree.** If you don't
      know why, ask it — then decide for yourself whether it was right.
- [ ] **You could rewrite it from scratch if you had to.** Not quickly. But
      you understand the problem well enough that you could.

---

### The 30-second version

If you only have time for three:

1. **Read every `if` out loud** — logic inversions are silent and permanent.
2. **Search the repo for what it just built** — it can't see your codebase.
3. **Explain the diff to yourself without the AI** — if you can't, don't merge.

---

*Free and MIT-licensed. Use it, fork it, put it in your team's PR template.*

*This checklist is the free piece of **The AI Code Review Kit** — which adds
the failure atlas, ~25 adversarial review prompts that make the AI attack its
own output, a drop-in `REVIEW.md` for Claude Code and Cursor, a context
protocol, and an automated pre-commit gate.*
