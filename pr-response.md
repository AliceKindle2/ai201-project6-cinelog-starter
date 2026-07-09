# PR Response Doc — CineLog Watchlist Feature

## AI Usage
<!-- Fill in at the end — how you used AI tools during this project -->

## Comment 1 — Rename
**What I did:** I changed save_to_watchlist to add_to_watchlist, also fixed routes/watchlist/watchlist.py for add_to_watchlist
**How I verified:** I booted up python app.py and verified the change was made. 

## Comment 2 — Deduplication
**What I did:** I followed the logic in collection_service duplication logic for add_to_collection and copied it for add_to_watchlist and made sure it worked. 
**How I verified:** I botted up python app.py and varified you can't make a duplication for the same film.

## Comment 3 — Missing test
**What I did:** I created a test_watchlist.py for watchlist_service following the same logic in collection_service.py
**How I verified:** I ran test_watchlist.py and varified that it's working correctly. 

## Comment 4 — Default visibility
**My position:** I am having public=true for people to be open and share the films they watched and going to watch. 
**Reasoning:** I'm optimizing for the watchlist to actually function as a shared feature from day one, not just a personal bookmark list. If entries default to private, the "public watchlist" surface (whatever browse/friends view consumes it) launches empty — nobody sees anybody else's list unless every single user finds and flips a toggle first, which most users never do. A True default means the feed has real content immediately, new users get value out of the social feature without needing to discover a setting, and "sharing what I want to watch" becomes the normal, expected behavior rather than an opt-in edge case. This also matches how most watchlist-style products behave — visibility is the default state, and privacy is the deliberate exception a user reaches for when they specifically want to hide something.
**Tradeoff acknowledged:**  The cost is consent by omission — a user who adds a film impulsively, or adds something they'd rather not broadcast, is exposed by default until they learn the toggle exists. That's a real privacy cost, and it lands hardest on new users who haven't yet discovered the setting. Mitigating that (a first-run notice, a visible toggle right on the add action) is worth doing, but as a default I'm choosing to prioritize making the social feature actually work over protecting against that edge case.

## Comment 5 — Sort order
**My position:** I am having it sort by date_added descending, not alphabetical.
**Reasoning:** A watchlist is a queue of intent, not a reference catalog — the question users ask is "what did I just add?", which recency answers directly and alphabetical order scatters.
**Engagement with reviewer's point:**  The maintainer's argument is that date-added matches how people actually use a watchlist, and I agree that's stronger than my original reasoning for alphabetical, which was really about making test output easy to scan, not about user behavior. The one case alphabetical helps — searching a long list by title — is better solved later with a filter/secondary sort than by making the default view work against the common case now.

## Comment 6 — Rebase
**What conflicted:** Nothing conflicted in the system.
**How I resolved it:** Nothing happened so there was nothing to resolve.
**How I verified no conflict remains:**  Everything is working as intended. 

## PR Description
<!-- Written at the end — feature overview, design decisions, manual testing steps -->