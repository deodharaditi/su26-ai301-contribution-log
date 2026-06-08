# Contribution 1: More control over ProgressBar formatting

**Contribution Number:** 1  
**Student:** Aditi Deodhar  
**Issue:** https://github.com/pytorch/ignite/issues/3118  
**Repository:** [pytorch/ignite](https://github.com/pytorch/ignite) — high-level training library for PyTorch (4.7k⭐, actively maintained)  
**Status:** Phase I — In Progress (posting approach-proposal comment to claim the issue; awaiting maintainer 👍 before coding)

---

## Why I Chose This Issue

I'm focused on AI/ML for this program, and PyTorch-Ignite is a well-known, actively maintained library in the PyTorch ecosystem — contributing here is both on-theme and a strong portfolio signal. The issue is small and well-bounded (improving formatting options on the `ProgressBar` handler), which fits a 3–4 week cycle and lets me get a real PR merged rather than getting lost in a large refactor.

It also matches my skills: it's pure Python and the change is localized to the progress-bar handler, so I can ramp up quickly with Claude Code rather than needing deep prior knowledge of the whole codebase. I'm hoping to learn how a mature open-source project structures its handlers and tests, how `tqdm` formatting works under the hood, and how to write a contribution that meets a high bar for code style (the project uses `ruff` + `pre-commit`) and test coverage.

---

## Understanding the Issue

### Problem Description

The `ProgressBar` handler doesn't expose enough control over how the bar is formatted. In particular, when you attach progress bars to multiple loops (e.g. a training loop and a separate validation/evaluation loop), the bars don't line up — the description and counter columns are different widths, so the actual `|bar|` sections start and end at different horizontal positions and the output looks misaligned.

### Expected Behavior

A user should be able to configure the bar's format (for example, pass a `tqdm` format string like `"{desc:<15} [{n_fmt}/{total_fmt}]|{bar}| {rate_fmt}{postfix}"`) so that multiple progress bars align cleanly, with consistent column widths and how the epoch/iteration info is displayed.

### Current Behavior

Formatting is largely fixed/internal — the `_OutputHandler` controls how epoch info is rendered and there's no clean public way to override the layout, so multiple bars render with mismatched widths.

### Affected Components

- `ignite/contrib/handlers/tqdm_logger.py` (the `ProgressBar` handler and its internal `_OutputHandler`) — _to confirm exact path/module during reproduction in Phase II._

### Maintainer Context & Prior Attempts

This issue is well-discussed, which is a strong signal for Phase II:

- **Maintainer engagement:** Collaborator `@vfdev-5` responded, invited a PR, and posted a workaround showing that careful `bar_format` strings + a mid-sized fill character can visually align bars — but confirmed there's currently no clean way to control *whether/how* epoch info is shown.
- **Original author's preferred design:** `@jmg049` (issue author) prototyped a `progress_with=<trainer_pbar>` parameter so a secondary (evaluator) bar inherits the trainer's epoch context, plus a custom `epochs` tqdm field. This linking approach — not just more format strings — is what the discussion converged toward.
- **Failed attempt to learn from — PR [#3761](https://github.com/pytorch/ignite/pull/3761):** A prior contributor added `show_epoch` (bool) + `epoch_format` (string). `@vfdev-5` **closed it unmerged**, saying it "does not address the issue and does wrong things." Two takeaways I'm carrying forward: **(1) agree on the approach in the issue thread before writing code, and (2) verify and locally test any AI-assisted code before opening a PR.**

---

## Reproduction Process

### Environment Setup

[Notes on setting up your local development environment - challenges you faced, how you solved them]

### Steps to Reproduce

1. [Step 1]
2. [Step 2]
3. [Observed result]

### Reproduction Evidence

- **Commit showing reproduction:** [Link to commit in your fork]
- **Screenshots/logs:** [If applicable]
- **My findings:** [What you discovered during reproduction]

---

## Solution Approach

### Analysis

[Your analysis of the root cause - what's causing the issue?]

### Proposed Solution

[High-level description of your fix approach]

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** [Restate the problem]

**Match:** [What similar patterns/solutions exist in the codebase?]
- Prior art: PR [#3761](https://github.com/pytorch/ignite/pull/3761) (`show_epoch` + `epoch_format`) was rejected — avoid that shape. Lean toward `@jmg049`'s `progress_with=` bar-linking idea and confirm scope with `@vfdev-5` first.
- Study how `_OutputHandler` in `tqdm_logger.py` currently injects epoch info, and how existing `ProgressBar` kwargs (e.g. `bar_format`, `persist`) are threaded through to `tqdm`.

**Plan:** [Step-by-step implementation plan]
1. [Modify file X to do Y]
2. [Add function Z]
3. [Update tests]

**Implement:** [Link to your branch/commits as you work]

**Review:** [Self-review checklist - does it follow the project's contribution guidelines?]

**Evaluate:** [How will you verify it works?]

---

## Testing Strategy

### Unit Tests

- [ ] Test case 1: [Description]
- [ ] Test case 2: [Description]
- [ ] Test case 3: [Description]

### Integration Tests

- [ ] Integration scenario 1
- [ ] Integration scenario 2

### Manual Testing

[What you tested manually and results]

---

## Implementation Notes

### Week [X] Progress

[What you built this week, challenges faced, decisions made]

### Week [Y] Progress

[Continue documenting as you work]

### Code Changes

- **Files modified:** [List]
- **Key commits:** [Links to important commits]
- **Approach decisions:** [Why you chose certain approaches]

---

## Pull Request

**PR Link:** [GitHub PR URL when submitted]

**PR Description:** [Draft or final PR description - much of the content above can be adapted]

**Maintainer Feedback:**
- [Date]: [Summary of feedback received]
- [Date]: [How you addressed it]

**Status:** [Awaiting review / Iterating / Approved / Merged]

---

## Learnings & Reflections

### Technical Skills Gained

[What you learned technically]

### Challenges Overcome

[What was hard and how you solved it]

### What I'd Do Differently Next Time

[Reflection on your process]

---

## Resources Used

- [Link to helpful documentation]
- [Tutorial or Stack Overflow post that helped]
- [GitHub issues or discussions that helped]
