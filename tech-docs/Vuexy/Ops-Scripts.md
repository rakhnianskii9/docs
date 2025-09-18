# Operational Scripts — User Discovery & Cleanup

These scripts help operators diagnose and clean Facebook-related users across both databases.

## Files
- scripts/find-user.sh
- scripts/clean-user.sh
- package.json scripts

## package.json entries
- user:find, find-user, fond-user — run find-user.sh
- user:clean, clean-user — run clean-user.sh

Usage examples:
- pnpm find-user (or pnpm user:find)
- pnpm clean-user (or pnpm user:clean)

## find-user.sh
- Purpose: display all data about a Facebook user across Vuexy and Flowise DBs.
- Writes a Markdown report under user-export/ with a timestamp (Asia/Ho_Chi_Minh) and auto-incremented filename.
- Also prints human-readable output to the console.
- Uses environment: DB_HOST/USER/PASSWORD and DB names (vuexy, flowise by default in the script).

Output highlights:
- Queries important tables and prints rows; also appends to the report file.

## clean-user.sh
- Purpose: fully remove a user and all related data from both DBs.
- Strict bash mode with safe defaults; uses psql with pager disabled for predictable non-interactive behavior.
- Environment: same as above. Review before running in production.

Safety:
- Always validate user identity (id, provider account id, pseudo-email) before deletion.
- Prefer taking a DB backup snapshot before running.

## Notes
- The scripts assume local Postgres access and credentials as set within the files. Parameterize as needed per environment.
