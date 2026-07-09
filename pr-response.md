# PR Response Doc — CineLog Watchlist Feature

## AI Usage
I used an AI assistant (Claude) to help me fix the models.py since it seemed to be having issues and stress test my logic for Comment 4 and Comment 5 to make sure it makes sense in the logic of watchlist_service.py and collection_service.py. I also had it check my commit history to make sure it holds up to standards in codepath. 

## Comment 1 — Rename
**What I did:** Renamed `save_to_watchlist` to `add_to_watchlist` for naming consistency with `add_to_collection`. I searched the codebase for every call site of the old name and found two: the definition in `services/watchlist_service.py` and the route handler in `routes/watchlist/watchlist.py`, which called it directly. I updated boty h.
**How I verified:** After renaming, I ran the full test suite (`python -m pytest tests/ -v`) to confirm nothing still referenced the old name — a leftover reference would have surfaced immediately as an `AttributeError` or `ImportError` at collection time. I also booted the app with `python app.py` and manually hit the watchlist route to confirm the renamed function is what's actually wired up, not just present in the file.

## Comment 2 — Deduplication
**What I did:** `add_to_collection` in `collection_service.py` prevents duplicate entries by querying for an existing `CollectionEntry` with the same `user_id` and `film_id` before inserting, and raising a custom exception if one is found. I mirrored that pattern in `add_to_watchlist`: before creating a new `WatchlistEntry`, it queries `WatchlistEntry.query.filter_by(user_id=user_id, film_id=film_id).first()`. If a match exists, it raises `AlreadyInWatchlistError` instead of inserting a second row — so a user can't end up with the same film on their watchlist twice.
**How I verified:** I booted the app with `python app.py` and manually called `add_to_watchlist` twice with the same `user_id`/`film_id` pair, confirming the second call raised `AlreadyInWatchlistError` rather than silently creating a duplicate row. This is also covered by an automated test (see Comment 3).

## Comment 3 — Missing test
**What I did:** Created `tests/test_watchlist.py`, modeled directly on the existing `tests/test_collection.py`. In particular, `test_add_to_watchlist_nonexistent_film_raises` mirrors `test_add_to_collection_nonexistent_film_raises` — it calls `add_to_watchlist` with a `film_id` that doesn't exist in the database (`"00000000-0000-0000-0000-000000000000"`) and asserts that `FilmNotFoundError` is raised, rather than letting a database integrity error surface. This targets the specific edge case the maintainer flagged: a caller passing a bad/nonexistent film reference should get a clear, catchable application error, not a raw DB exception.
**How I verified:** Ran `python -m pytest tests/test_watchlist.py -v` and confirmed the test passes, then ran the full suite (`python -m pytest tests/ -v`) to confirm it didn't break the existing collection tests.

## Comment 4 — Default visibility
**My position:** `public` should default to `True`.
**Reasoning:** I'm optimizing for the watchlist to function as a shared feature from day one, not just a personal bookmark list. If entries default to private, the "public watchlist" surface (whatever browse/friends view eventually consumes it) launches empty — nobody sees anybody else's list unless every single user finds and flips a toggle first, which most users never do. A `True` default means the feed has real content immediately, new users get value out of the social feature without needing to discover a setting, and "sharing what I want to watch" becomes the normal, expected behavior rather than an opt-in edge case reserved for power users. For a film-logging app like CineLog specifically, the whole point of the product is social/discovery-driven (seeing what friends are watching), so a watchlist that's invisible by default works against the app's core value proposition.
**Tradeoff acknowledged:** The cost is consent by omission — a user who adds a film impulsively, or adds something they'd rather not broadcast, is exposed by default until they learn the toggle exists. That cost lands hardest on new users who haven't yet discovered the setting. Mitigating that (a first-run notice, a visible toggle right on the add action) is worth doing, but as a *default*, I'm choosing to prioritize making the social feature actually work over protecting against that edge case.

## Comment 5 — Sort order
**My position:** Sort by `date_added` descending, not alphabetical.
**Reasoning:** A watchlist is a queue of intent, not a reference catalog. For CineLog specifically, the watchlist answers "what do I want to watch next / what did I just add," and recency answers that directly — alphabetical order scatters a newly added film wherever its title happens to fall, disconnecting it from the moment the user expressed interest in it.
**Engagement with reviewer's point:** The maintainer's argument is that date-added matches how people actually use a watchlist ("most users want to see what they added recently"), and I agree — it's stronger than my original reasoning for alphabetical, which was really about making test output easy to eyeball during development, not about user behavior. The one case alphabetical genuinely helps is a long list a user wants to search/browse by title (e.g., "did I already add that Kubrick film?"), but that's a search/filter problem, better solved later with an explicit title filter or secondary sort toggle, than by making the default view work against the common "what's new" case today.

## Comment 6 — Rebase
**What conflicted:** There was no git merge conflict during the rebase itself — all watchlist work was committed directly on top of the current `main`, so there was nothing for git to reconcile line-by-line. The real conflict was a *logical* one, introduced by timing: `watchlist_service.py` was originally written against a pre-refactor version of the schema, and its own docstring said as much (`film_id (int): ID of the film. (Note: integer — pre-refactor)`). By the time this code needed to run, `main` had already migrated `Film.id` (and every other model's primary/foreign keys) from integer to UUID strings (`db.String(36)`). That mismatch meant `WatchlistEntry` didn't exist yet with the right column types, and the service layer's assumptions no longer matched the rest of the schema.
**How I resolved it:** I added `WatchlistEntry` to `models.py` using `db.String(36)` for `id`, `user_id`, and `film_id`, with `user_id` and `film_id` as foreign keys to `user.id` and `film.id` respectively — matching the exact pattern `CollectionEntry` already uses post-refactor. I also removed the stale integer-ID assumption from `watchlist_service.py` and updated `test_watchlist.py` to use a UUID string (`"00000000-0000-0000-0000-000000000000"`) for the nonexistent-film test case instead of an integer.
**How I verified no conflict remains:** Ran the full test suite (`python -m pytest tests/ -v`) after the fix — all tests pass, including the watchlist tests that exercise `film_id` as a UUID. I also confirmed via `git log --oneline` that the branch history is linear with no `Merge branch` commits from this work (the one pre-existing merge commit, `bbe206c`, predates the watchlist feature and belongs to an unrelated, already-merged `.gitignore` PR).

## PR Description

**What this feature does:**
This PR adds a watchlist feature to CineLog. Users can save films they want to watch later (`add_to_watchlist`), preventing duplicate saves of the same film, and retrieve their full watchlist (`get_watchlist`) sorted with the most recently added film first. Each watchlist entry also carries a `public` flag so a user's watchlist can be shown on a shared/browse view.

**Design decisions:**
- **Default visibility:** `public` defaults to `True` — new watchlist entries are shareable by default, so the social/discovery feature has real content from day one rather than launching empty (see Comment 4 for full reasoning and the tradeoff).
- **Sort order:** watchlist entries are returned sorted by `date_added` descending (most recent first), matching how users actually think about a watchlist — "what did I just add" — rather than alphabetically (see Comment 5).

**How to manually test:**
1. Run `python app.py` to start the server.
2. Create or use an existing user and film record (via the existing collection endpoints/seed data).
3. Call the add-to-watchlist route with a valid `user_id` and `film_id` — confirm a new entry is created and returned.
4. Call it again with the same `user_id`/`film_id` — confirm it raises `AlreadyInWatchlistError` instead of creating a duplicate.
5. Call it with a `film_id` that doesn't exist in the `film` table — confirm it raises `FilmNotFoundError` rather than a raw database error.
6. Call the get-watchlist route for that user — confirm the film appears, and that adding a second film causes the most recently added one to appear first in the returned list.
7. Run `python -m pytest tests/ -v` — all tests, including `tests/test_watchlist.py`, should pass.

![alt text](<Screenshot 2026-07-09 101159.png>)