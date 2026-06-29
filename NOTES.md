# Notes

## Summary of changes
1. **SQL Logic Error (Backend)**: Fixed a bug in `TaskRepository.java` where the `OR` condition for title and description filtering lacked parentheses. This was causing incorrect precedence when evaluated with `AND` conditions.
2. **Artificial Delay (Backend)**: Removed an intentional performance bottleneck in `TaskController.java` (`Thread.sleep(queryWeight)`) which slowed down searches artificially.
3. **Frontend Debounce & Pagination Reset**: Added a 300ms debounce to the search query in `App.jsx` to prevent excessive API requests while typing. Also updated the search and status filter handlers to reset the pagination to page 1 whenever filters change.

## What I chose not to change and why
I left the pagination logic in `TaskController.java` as in-memory list subsetting (`allResults.subList`). While a database-level limit/offset (`Pageable` in Spring Data JPA) is far more scalable and performant, the exercise explicitly prohibited a large rewrite and prioritizing focused diffs. Given it's an H2 in-memory DB and the problem scope, the subsetting isn't functionally breaking, just unscalable. I chose to fix the functional SQL logic error and the Thread.sleep bottleneck which had immediate user-facing impacts instead.

## The biggest remaining risk
The biggest risk is the lack of proper database-level pagination. The `TaskRepository` fetches *all* non-archived tasks matching the search criteria into Java memory, then the controller trims the list. As the dataset grows, this will cause severe memory and performance issues (OutOfMemoryError) because the entire result set is loaded into heap memory before being paginated.

## What tools/AI you used
Used an agentic coding assistant to clone the repository, explore the file structure, and read code. Identified the issues by reading the code. Used code editing tools to apply the precise focused changes directly to the respective files. Generated handwritten notes programmatically.
