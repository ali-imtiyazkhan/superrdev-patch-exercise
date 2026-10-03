# Notes

## Summary of changes

I fixed five things:

1. **Removed the `Thread.sleep()` in `TaskController.java`.** It slept up to 1 second per request as a fake "complexity score". This was the first thing I hit — typing in the search box felt sluggish — so I checked the controller and found it blocking the request thread. Deleted the block.
2. **Fixed the SQL in `TaskRepository.java`.** `AND` binds tighter than `OR`, so `archived = FALSE AND title LIKE … OR description LIKE … AND status = …` was really `(archived AND title) OR (description AND status)`. Archived tasks leaked into results and the status filter was ignored for title matches. Wrapped the two `LIKE` clauses in parentheses; mirrored the same fix in `db/queries/search_tasks.sql` and both queries in the Oracle package.
3. **Fixed stale-response races in `useTasks.js`.** Added an `AbortController` cleanup, a 300 ms debounce, and made sure `error` clears on success and `loading` clears on failure (it used to hang on "Loading..." after one error).
4. **Reset pagination to page 1** when the search term or status changes in `App.jsx` — otherwise you could sit on an out-of-range page showing nothing. Also replaced the hardcoded `10` with a `PAGE_SIZE` constant.
5. **Validated controller input** — an unknown `status` returns 400 instead of a 500, and `page`/`pageSize` are clamped so `subList` can't throw.

## What I didn't change

Pagination still loads every matching row and slices in memory. It's wasteful, but moving it into SQL changes the repository contract and the Oracle artifact too — too much churn for a patch. The UI also doesn't escape `%`/`_` in search terms.

## Biggest remaining risk

In-memory pagination. Fine at ~50 rows, ugly at 50k.

## Tools

Used Claude to read the codebase and draft fixes; I picked the order myself, checked every diff, and rewrote parts of this file by hand.
