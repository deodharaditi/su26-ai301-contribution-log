# Contribution 1: Document authentication via fsspec `storage_options`

**Contribution Number:** 1  
**Student:** Aditi Deodhar  
**Issue:** https://github.com/zarr-developers/zarr-python/issues/2995  
**Repository:** [zarr-developers/zarr-python](https://github.com/zarr-developers/zarr-python) — Python library for chunked, compressed N-dimensional arrays, widely used in ML/scientific data pipelines (2k⭐, actively maintained)  
**Status:** Phase II — Complete (env set up, gap reproduced, branch created, UMPIRE solution plan written) → ready to implement in Phase III

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

**OS:** Windows 11 · **Tooling found:** Anaconda, Git, VS Code. The project has **no dev container**, so this was the "typical case" (follow README/CI setup). Docs are built with **MkDocs** (`mkdocs.yml`), and `.python-version` pins **Python 3.12**; the docs build recipe comes from `.readthedocs.yaml` (`pip install .[remote] --group docs`).

Steps that worked:

```powershell
# 1. Create a clean Python 3.12 env (see gotcha #1 below)
conda create -n zarr-dev python=3.12 -y

# 2. Fork on GitHub, then clone the fork + add upstream
git clone https://github.com/deodharaditi/zarr-python.git
cd zarr-python
git remote add upstream https://github.com/zarr-developers/zarr-python.git

# 3. Upgrade pip (see gotcha #2), then editable install with remote + docs deps
conda run -n zarr-dev python -m pip install --upgrade pip
conda run -n zarr-dev python -m pip install -e ".[remote]" --group docs

# 4. Verify, then build/preview the docs
conda run -n zarr-dev python -c "import zarr, s3fs, fsspec; print(zarr.__version__)"
conda run -n zarr-dev mkdocs build        # one-shot build (~74s)
conda run -n zarr-dev mkdocs serve        # live preview at http://127.0.0.1:8000
```

**Challenges & fixes (for future contributors):**
1. **Stale Python launcher:** `py -3.12` was registered to `C:\Python312\python.exe`, which didn't exist (`The system cannot find the file specified`). Fixed by ignoring the system launcher and creating a dedicated `conda` env pinned to Python 3.12.
2. **`--group docs` failed on old pip:** the `--group` flag (PEP 735 dependency groups) requires **pip ≥ 25.1**. Anaconda's bundled pip was older, so `pip install --upgrade pip` was required first.
3. **Misleading build warning:** `mkdocs build` prints a loud banner claiming "MkDocs is abandoned, switch to ProperDocs." This is a promotional message injected by a third-party plugin, **not** a real problem — the build completes successfully. Ignored it (can be silenced with `DISABLE_MKDOCS_2_WARNING=true`).

Result: `import zarr` works (editable dev build `0.1.dev2913`), and the docs site builds locally in ~74s — environment is ready.

### Steps to Reproduce

This is a documentation gap (not a runtime bug), so "reproduction" = confirming the gap exists in the current docs:

1. Open `docs/user-guide/storage.md` (renders at the storage user-guide page) and read the **Remote Store** section.
2. Observe that every remote/S3 example uses **anonymous access** (`storage_options={'anon': True}`) or `client_kwargs={'endpoint_url': ...}`.
3. Note there is **no section covering credential-based authentication** (access key + secret) for private S3 / MinIO buckets — which is exactly what the issue reporter needed.

### Branch Link

- Working branch: [`docs-issue-2995`](https://github.com/deodharaditi/zarr-python/tree/docs-issue-2995) (created from up-to-date `main`, pushed to my fork)

### Reproduction Evidence

- **Source file confirmed:** [`docs/user-guide/storage.md`](https://github.com/zarr-developers/zarr-python/blob/main/docs/user-guide/storage.md) — the `### Remote Store` section (lines ~120–152) only shows anonymous (`anon: True`) examples.
- **Canonical auth keys confirmed locally:** I inspected the installed `s3fs` (v2026.4.0) in my dev env:
  ```python
  import s3fs, inspect
  print(inspect.signature(s3fs.S3FileSystem.__init__))
  # -> anon, endpoint_url, key, secret, token, use_ssl, client_kwargs, username, password, ...
  ```
  So `key`, `secret`, `token`, and `endpoint_url` are **top-level** `S3FileSystem` parameters that `fsspec` consumes from `storage_options`.
- **Root cause of the reporter's error:** `TypeError: ClientSession._request() got an unexpected keyword argument 'secret'` occurs when credentials are nested inside `client_kwargs`. `client_kwargs` is forwarded to the underlying `aiohttp`/`aiobotocore` `ClientSession`, which doesn't accept `secret` — so credentials must go at the **top level** of `storage_options`, not inside `client_kwargs`. This is precisely the distinction the docs never state.

---

## Solution Approach

### Analysis

The root cause is a **documentation gap**, not a code bug. `FsspecStore.from_url(..., storage_options=...)` passes `storage_options` straight to the fsspec backend (`s3fs.S3FileSystem`). The credential parameters (`key`, `secret`, `token`, `endpoint_url`) are top-level `S3FileSystem` arguments, but the storage guide only ever demonstrates anonymous access, so users guess — and a common wrong guess (nesting credentials in `client_kwargs`) routes them to the aiohttp `ClientSession`, producing the cryptic `unexpected keyword argument 'secret'` error.

### Proposed Solution

Add a short **"Authentication"** subsection to the `### Remote Store` section of `docs/user-guide/storage.md` that shows how to pass credentials for a private S3 / MinIO bucket via `storage_options`, and explicitly calls out that credentials go at the **top level** (not inside `client_kwargs`). Because the docs build **executes** code blocks (`markdown-exec`), the credential example must be a **non-executed** `python` block (we can't connect to a private bucket in CI).

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** Zarr's storage docs don't explain how to authenticate to private remote stores; only anonymous examples exist. Users need to know which `storage_options` keys carry credentials and where they belong.

**Match:**
- Mirror the existing `### Remote Store` examples in `storage.md` (lines ~120–152) — same `FsspecStore.from_url(..., storage_options={...})` shape and prose style.
- Ground the keys in the verified `s3fs.S3FileSystem` signature (`key`, `secret`, `token`, `endpoint_url`, `anon`).

**Plan:**
1. In `docs/user-guide/storage.md`, immediately after the Remote Store examples (before `### Memory Store`), add an **"Authentication"** subsection.
2. Add a **non-executed** `python` code block showing a private-bucket / MinIO example:
   `storage_options={'key': '...', 'secret': '...', 'endpoint_url': 'https://...'}` — plus notes on `token` (temporary creds) and `anon=True` (public).
3. Add a callout: **credentials go at the top level of `storage_options`, not in `client_kwargs`** (which is forwarded to aiohttp and causes the reporter's error). Link to the `s3fs`/`fsspec` docs.
4. Add a towncrier news fragment: `changes/2995.doc.md`.

**Implement:** _(Phase III)_ on branch [`docs-issue-2995`](https://github.com/deodharaditi/zarr-python/tree/docs-issue-2995).

**Review:** Run the `prek` pre-commit hooks (project uses `prek`, compatible with `.pre-commit-config.yaml`); rebase on `upstream/main` before opening the PR; follow the repo's `PULL_REQUEST_TEMPLATE.md` and reference issue #2995; keep the change scoped to docs only.

**Evaluate:** This is a docs-only change, so no unit tests are required (code coverage is unaffected). Verification:
1. Build the docs locally with `mkdocs build --strict` and confirm **no new warnings/broken cross-references**.
2. `mkdocs serve` and visually confirm the new **Authentication** subsection renders correctly on the storage page.
3. Confirm the towncrier fragment is picked up (`towncrier build --draft`).
4. Have a knowledgeable reader sanity-check that the documented `key`/`secret`/`endpoint_url` placement is correct against the `s3fs` API.

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
