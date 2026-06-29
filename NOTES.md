# Notes

## Summary of Changes

**Bug 1 — SQL OR Precedence (Critical, Backend)**
Fixed in `TaskRepository.java`, `db/queries/search_tasks.sql`, and `db/oracle/task_search_package.sql`.
The `WHERE` clause had `AND ... OR ... AND` without parentheses around the OR. Due to SQL operator precedence (AND binds tighter than OR), the `archived = FALSE` and `status` filters were only applied to the `title` branch, not the `description` branch. This meant archived tasks could leak through and the status filter was silently ignored for description matches. Fixed by wrapping the OR in parentheses: `AND (title LIKE :term OR description LIKE :term)`.

**Bug 2 — Artificial Latency (High, Backend)**
Removed from `TaskController.java`. A `Thread.sleep()` block was intentionally injecting up to 1000ms of delay per request, inversely proportional to the search query length. Short/empty queries would block a thread for a full second on every keystroke. Removed entirely.

**Bug 3 — Loading State Not Cleared on Error (Medium, Frontend)**
Fixed in `useTasks.js`. `setLoading(false)` was called only in `.then()` but not in `.catch()`. On any API error, the loading spinner would stay visible forever — the error message would never render.

**Bug 4 — No Search Debounce / No Pagination Reset (Medium, Frontend)**
Fixed in `App.jsx`. Added a 300ms debounce before the query is sent to the hook, reducing API hammering while the user types. Also reset `page` to `1` whenever query or status filter changes, preventing empty result pages.

## What I Chose Not to Change
In-memory pagination in `TaskController.java`: the controller loads all matching rows into Java heap and slices the list. Replacing this with `Pageable` in Spring Data JPA would be the right fix, but it requires changes across the repository interface and controller, which goes beyond a focused patch.

## Biggest Remaining Risk
The in-memory pagination is the biggest scaling risk. As the dataset grows, fetching all rows into memory before subsetting will cause heap exhaustion. This should be replaced with database-level `LIMIT`/`OFFSET` (Spring Data `Pageable`).

## Tools / AI Used
Used an agentic AI coding assistant (Antigravity / Gemini) to clone the repo, navigate the file structure, and make precise file edits. Independently reviewed each diff and verified correctness before committing. All bug identification and reasoning is my own.
