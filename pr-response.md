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
**My position:**
**Reasoning:**
**Tradeoff acknowledged:**

## Comment 5 — Sort order
**My position:**
**Reasoning:**
**Engagement with reviewer's point:**

## Comment 6 — Rebase
**What conflicted:**
**How I resolved it:**
**How I verified no conflict remains:**

## PR Description
<!-- Written at the end — feature overview, design decisions, manual testing steps -->