# Contribution 1: Document authentication via fsspec `storage_options`

**Contribution Number:** 1  
**Student:** Aditi Deodhar  
**Issue:** https://github.com/zarr-developers/zarr-python/issues/2995  
**Repository:** [zarr-developers/zarr-python](https://github.com/zarr-developers/zarr-python) — Python library for chunked, compressed N-dimensional arrays, widely used in ML/scientific data pipelines (2k⭐, actively maintained)  
**Status:** Phase I — In Progress (preparing to claim the issue)

---

## Why I Chose This Issue

Zarr is a pure-Python library that's heavily used in ML and scientific data pipelines for storing large, chunked arrays, so contributing here is relevant to the data side of AI/ML work. I deliberately chose a well-scoped documentation issue for my first contribution: it's recent (opened April 2025), unassigned, has no competing pull request, and the reporter already included a working code example, so the path to a merged PR is clear rather than open-ended.

It matches my skills and learning goals: the work is Python documentation plus light research into how `fsspec` / `s3fs` authentication parameters flow through `storage_options`. I'm hoping to learn how a mature open-source project structures its docs and contribution workflow, get comfortable with its `pre-commit`/CI checks, and land a real merged PR I can build on for a more code-heavy second contribution.

---

## Understanding the Issue

### Problem Description

Zarr's documentation doesn't explain which authentication keys are valid inside the `storage_options` dict when opening a store backed by `fsspec` (e.g. S3 / MinIO). Users guess at keys like `key`/`secret` or `username`/`password`, and when they're wrong they get cryptic errors such as `TypeError: ClientSession._request() got an unexpected keyword argument 'secret'`.

### Expected Behavior

The storage user-guide should clearly document how to authenticate to common `fsspec` backends via `storage_options` (which keys are accepted for S3/HTTP, or a pointer to the relevant `fsspec`/`s3fs` configuration), ideally with a short working example.

### Current Behavior

The storage guide covers opening remote stores but omits an authentication section, leaving users to trial-and-error the `storage_options` keys.

### Affected Components

- The storage user-guide page (`zarr.readthedocs.io/en/latest/user-guide/storage.html`) — corresponding source likely under `docs/user-guide/storage.rst` (_to confirm exact path during Phase II_).
- Behavior is delegated to `fsspec` / `s3fs`, so the fix is documentation pointing at the correct backend auth parameters, not a code change.

### Maintainer Context & Hints

- The issue reporter included the exact failing error **and** a working solution using `s3fs` directly with the proper parameters — so there's a concrete, tested example to base the docs on.
- Labels: `documentation`, `help wanted`. Unassigned, no linked PR, 0 comments as of selection — low contention, but I'll still post a short comment to claim it and confirm the doc location/scope with maintainers before writing.

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
- Find how the existing storage docs are structured (`docs/user-guide/storage.*`) and mirror that style for a new auth section.
- Base the example on the reporter's working `s3fs` snippet; cross-check against `fsspec`/`s3fs` docs for the canonical `storage_options` auth keys (e.g. `key`, `secret`, `endpoint_url`, `client_kwargs`).

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
