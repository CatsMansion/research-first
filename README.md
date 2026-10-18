# research-first

**Do research with stopping criteria instead of vibes.**

Most agents stop researching when they *feel* they have enough. That stopping signal is
not reproducible, not transferable, and not auditable — and it silently produces wrong
answers that *look* researched.

This skill turns "I think that's enough" into machine-checkable criteria, and ships two
CLI checkers that will happily report **fail** on a journal you wrote yourself.

---

## Why this exists

The failure mode is not "searched too little". It's **not knowing when to stop** — and
not knowing what you didn't think to look for. Both are invisible from inside the work.

Concretely, this skill was written after its own author shipped two confidently wrong
claims to a spec that other people were building against. Both errors were found by a
*later* cross-check, not by reading harder:

| Wrong claim | What it should have been |
|---|---|
| "impact frames must **not** be a white flash" | White / inverted high-contrast **is** the standard form |
| "disable motion blur" | That kills `smear frames`, a legitimate 2D technique; only *procedurally generated* blur is the problem |

What was missing was not knowledge. It was **absence of a stopping criterion** and
**absence of a failure-case search**. That is what this skill is for.

---

## What it gives you

### 1. Five stopping criteria (S1–S5)

| # | Criterion | Threshold |
|---|---|---|
| **S1** | Independent sources — not reprints, not summaries of one doc | ≥3 distinct domains |
| **S2** | Every conclusion traceable to `URL + section`; otherwise marked **unverified** | 0 untraceable-and-unmarked |
| **S3** | Marginal convergence — stop when 2 consecutive searches add nothing new | 2 consecutive zero-yield rounds |
| **S4** | ⛔ Failure cases searched ("why does this go wrong") | section present with real content |
| **S5** | Anything quantifiable carries a **number + source**; no bare adjectives | 0 unsourced numbers |

### 2. Six breadth axes

More research comes from **switching axes**, not from searching more:
source count · language (use the domain's **native** terms) · time span ·
⛔ failure cases · quantification · local empirical measurement.

### 3. Nine associative-expansion directions (D1–D9)

Guards against the most expensive omission — never thinking to look.

`D1` confusable neighbours · `D2` first-hand origin · **`D3` who profits from the claim** ·
`D4` prerequisites · `D5` failure modes · `D6` quantified parameters · `D7` alternatives ·
**`D8` cross-domain/cross-era differences** · `D9` primary sources

`D3` and `D8` are the two that most often surface something nobody was looking for.

### 4. A journal template

Conclusions carry a source column; withdrawals are recorded, not silently deleted.

---

## Install

```bash
# Vercel skills CLI — auto-detects your agents and writes/symlinks into place
npx skills add <owner>/<repo>

# or copy the folder directly
cp -r research-first ~/.claude/skills/
```

---

## Usage

Write your research journal following `references/纪要模板.md`, then:

```bash
python scripts/run_all.py journal.md            # both checkers, one command
python scripts/check_journal.py journal.md      # S1–S5: are the conclusions sound?
python scripts/check_expansion.py journal.md    # D1–D9: did I only look one direction?
```

Exit codes: `0` pass · `1` red items · `2` parse failure (guards against *vacuous pass*).

⚠️ **Exit 0 does not mean perfect.** `WARN` items are not failures but mean something:

- **S3 WARN** — not yet converged; don't call it "finished"
- **S4 WARN** — real content, but still has open gaps
- **D ⚠️** — found *passively*, not by active expansion

---

## Design notes

**Machine-checkable, not self-reported.** Both checkers must be run against a journal the
author wrote themselves and must be willing to report red. Both refuse to report "pass"
when they parsed zero rows — otherwise a format change turns the criterion into a rubber stamp.

**Anti-vacuous-pass.** Three bugs were found and fixed in the checkers themselves, all of the
same family — *a checker that reports green when it shouldn't*:

1. Substring table-header matching (`'结论' in line`) matched ordinary prose rows
2. Not stripping ``` code fences, so template examples inside the journal were read as real data
3. **Index-vs-line-number confusion** — `strip_fences()` returns `(lineno, text)`, and the
   code sliced by `range(i+1, …)` as if `i` were a list index, silently reading a section
   20 lines away. This produced a **false negative** (judged a "not done" section as done).

Rule learned: **when a checker has been wrong three times, stop reading the code and print
the intermediate values.**

**Word counts, not keyword hits.** An early version decided "did they search failure cases?"
by testing for the presence of the word "not done". After that section was properly filled in,
a disclaimer sentence still contained the phrase — and the checker kept reporting red.
It now measures **net content** (negation sentences removed) instead.

---

## Compatibility

Python 3.8+, standard library only. No network calls, no external packages.

Follows the [Agent Skills](https://agentskills.io) open standard: directory name matches
`name` in frontmatter; `SKILL.md` stays under 500 lines; heavy content in `references/`,
executables in `scripts/`.

## License

MIT
