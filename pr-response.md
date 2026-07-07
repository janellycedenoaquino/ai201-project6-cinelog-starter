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

**What I did:** Added deduplication logic to `add_to_watchlist()` in `services/watchlist_service.py`, mirroring the pattern in `add_to_collection()`: after confirming the film exists, query for an existing `WatchlistEntry` matching `user_id` + `film_id` via `.filter_by(...).first()`, and raise before inserting if one is found. Defined a new `AlreadyInWatchlistError` exception in `watchlist_service.py` rather than reusing `collection_service.py`'s `AlreadyInCollectionError`, since the two errors represent different domains (collection vs. watchlist) and a caller should be able to distinguish them.

**How I verified:** Initially reused `AlreadyInCollectionError` imported from `collection_service.py` to raise on a watchlist duplicate — on review, caught that this conflated two domains under one exception class and would prevent callers from telling a collection duplicate apart from a watchlist duplicate. Replaced it with a locally-defined `AlreadyInWatchlistError`, matching the naming convention of `collection_service.py`'s own exception classes. Ran `pytest tests/ -v` after the change to confirm the existing 4 tests still pass (no `test_watchlist.py` yet — that's Comment 3).

## Comment 3 — Missing test
> Please add a test for the case where `film_id` doesn't exist in the database. Look at the existing tests in `test_collection.py` — the pattern is there.
> — @dev-lead, general PR comment

**What I did:** Created `tests/test_watchlist.py` with `test_add_to_watchlist_nonexistent_film_raises`, modeled directly on `test_add_to_collection_nonexistent_film_raises` in `test_collection.py`. Reused the same `app` and `sample_user` fixture structure (in-memory SQLite, created/torn down per test) and the same fake-UUID approach (`"00000000-0000-0000-0000-000000000000"`) to assert `add_to_watchlist()` raises `FilmNotFoundError` for a film that doesn't exist. No `sample_film` fixture needed, since the collection test it's modeled on doesn't use one either — the point of the test is the nonexistent case.

**How I verified:** Ran `pytest tests/test_watchlist.py -v` to confirm the new test passes on its own, then `pytest tests/ -v` to confirm all 5 tests across both files pass together with no regressions.

## Comment 4 — Default visibility
> I notice watchlists default to `public=True`. We don't have a documented decision on default visibility for user lists. Before I can approve this, I need you to add a note to your PR description explaining your reasoning. I want to make sure we're being intentional here, not just inheriting a default.
> — @dev-lead, general PR comment

**My position:** I changed the default from `public=True` to `public=False`. Watchlist entries should be private unless a user explicitly chooses to share them.

**Reasoning:** Right now there is no way for a user to set `public` themselves — `POST /watchlist/<user_id>/add` only accepts `film_id`. That means the current `True` default isn't really a "default" in the normal sense of "the value most users would pick if asked" — it's the *only* outcome, applied to every entry, for every user, with no exceptions and no input from anyone. If we're going to force one outcome on all users until a visibility toggle exists, it should be the outcome that doesn't expose anything they didn't choose to expose. A watchlist can reveal more than someone intends to share — what they're planning to watch, when, and why — and none of that should become visible without some action on their part.

**Tradeoff acknowledged:** CineLog is a *community* film tracking app, and a private-by-default watchlist means the social/discovery value of watchlists (friends seeing what you want to watch) starts at zero engagement until users manually opt in — most users never change a default, so this could make any future "browse watchlists" feature launch quieter than it would with `public=True`. I'm accepting that tradeoff because the failure modes aren't symmetric: an overly private default just means slower adoption of a social feature, which is fully recoverable later with an onboarding prompt or UI nudge. An overly public default means real exposure — someone's data being visible before they ever decided that was okay — and that can't be undone once it's happened. I'd rather under-share by default and let users opt in than expose everyone by default and hope no one minds.

## Comment 5 — Sort order
> I'd prefer watchlists to default to "date added" order rather than alphabetical. Most users want to see what they added recently. I'm open to discussion if you see it differently — but let's make a decision and document it.
> — @dev-lead, `services/watchlist_service.py:50`

**My position:** I agree with the reviewer. Changed `get_watchlist()` in `services/watchlist_service.py` to sort by `WatchlistEntry.date_added.desc()` (newest first), replacing the previous `Film.title.asc()` (alphabetical).

**Reasoning:** `get_collection()` in `collection_service.py` already sorts the analogous "already watched" list by `date_added.desc()`. Watchlist and collection are the two most similar list views in the app, and having one sort by recency while the other sorts alphabetically was an inconsistency with no clear justification — a user familiar with how their collection displays would reasonably expect their watchlist to behave the same way.

**Engagement with reviewer's point:** The reviewer's argument was that most users want to see what they added recently, and I don't have a reason to disagree — if anything, the existing `get_collection()` precedent suggests the product already made this same UX bet once and it's worked. The main case for alphabetical (finding a specific title, or checking whether you already added something) is largely handled elsewhere now: Comment 2's deduplication logic already prevents duplicate entries at the service layer, so alphabetical order isn't doing much load-bearing work for that use case anymore. A future "let the caller choose sort order via a query param" option could still be worth exploring later (similar to how `routes/films.py` supports `?genre=` and `?year=` filters), but that's a bigger, separate feature than what this comment is asking for — a decision on the default.

## Comment 6 — Rebase
> A refactor merged to `main` that changed film IDs from integers to UUIDs. Your watchlist code still references integer IDs. Please rebase on `main` and update accordingly.
> — @dev-lead, general PR comment

**What conflicted:**
**How I resolved it:**
**How I verified no conflict remains:**

## PR Description
<!-- Written at the end — feature overview, design decisions, manual testing steps -->
