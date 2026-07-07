# PR Response Doc — CineLog Watchlist Feature

## AI Usage
<!-- Fill in at the end — how you used AI tools during this project -->

## Comment 1 — Rename
> `save_to_watchlist()` should follow the project's naming convention. Compare with `add_to_collection()` — the pattern here is `verb_to_noun`. Please rename to `add_to_watchlist()` and update all call sites.
> — @dev-lead, `services/watchlist_service.py:12`

**What I did:** Renamed `save_to_watchlist()` to `add_to_watchlist()` in `services/watchlist_service.py` to match the project's `verb_to_noun` convention used elsewhere (`add_to_collection()`, `remove_from_collection()`).

**How I verified:** Ran `grep -rn "save_to_watchlist" --include="*.py" .` across the repo before editing to find every reference (3 total: the definition, the import, and the call site in `routes/watchlist/watchlist.py`). Updated all three, then re-ran the same grep — zero matches for the old name. Ran `pytest tests/ -v` to confirm the existing test suite (4 tests) still passes.

## Comment 2 — Deduplication
> What happens if a user calls this with a film that's already on their watchlist? The current implementation would add a duplicate entry. Please handle this case.
> — @dev-lead, `services/watchlist_service.py:30`

**What I did:**
**How I verified:**

## Comment 3 — Missing test
> Please add a test for the case where `film_id` doesn't exist in the database. Look at the existing tests in `test_collection.py` — the pattern is there.
> — @dev-lead, general PR comment

**What I did:**
**How I verified:**

## Comment 4 — Default visibility
> I notice watchlists default to `public=True`. We don't have a documented decision on default visibility for user lists. Before I can approve this, I need you to add a note to your PR description explaining your reasoning. I want to make sure we're being intentional here, not just inheriting a default.
> — @dev-lead, general PR comment

**My position:**
**Reasoning:**
**Tradeoff acknowledged:**

## Comment 5 — Sort order
> I'd prefer watchlists to default to "date added" order rather than alphabetical. Most users want to see what they added recently. I'm open to discussion if you see it differently — but let's make a decision and document it.
> — @dev-lead, `services/watchlist_service.py:50`

**My position:**
**Reasoning:**
**Engagement with reviewer's point:**

## Comment 6 — Rebase
> A refactor merged to `main` that changed film IDs from integers to UUIDs. Your watchlist code still references integer IDs. Please rebase on `main` and update accordingly.
> — @dev-lead, general PR comment

**What conflicted:**
**How I resolved it:**
**How I verified no conflict remains:**

## PR Description
<!-- Written at the end — feature overview, design decisions, manual testing steps -->
