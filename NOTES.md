# NOTES

## What's in CLAUDE.md, and what I left out

I kept what Claude can't safely guess: the exact commands (including how to run one test file and that CI runs lint before tests), how requests flow from `server.js` through `routes/` to `db/store.js`, and rules that each answer a "which way?" question — CommonJS rather than ES modules, data access only through the store, one error-response shape with fixed status codes, and `node:test` + `supertest` rather than Jest. The note that the store is shared between tests is there because it's an easy way to write a flaky test.

I left out a file-by-file listing, explanations of how Express works, the endpoints themselves and the sample users, because Claude can read those from the code in seconds and a copy in CLAUDE.md would go stale. There are no secrets or values from `.env`, and no one-off tasks: every line should still be true and useful in the next session.

## Permission rules

- **Allow:** `npm test`, `npm run lint` and `node --test …`. They only read code and run checks, and Claude runs them after almost every change, so approving them each time adds nothing.
- **Ask:** `git push`, because it publishes work to GitHub and I want to see what's going out first.
- **Deny:** reading or editing `.env`, `git push --force` / `-f`, and `git reset --hard`.

Without the deny rules, Claude could open `.env` while exploring and copy a real secret into the conversation or a file, or rewrite the remote branch and throw away commits with a force-push, or discard uncommitted work with a hard reset. Deny rules win over allow and ask, so these stay blocked even if I approve something broader later.

## Verification

My own OS can't run Claude Code locally, so I ran these checks in Claude Code on the web, in a fresh session on this repo's `set-up-claude` branch. The web app's `/permissions` opens the permission-mode picker (Auto / Accept edits / Plan) rather than the rules list, so I checked the same things by asking Claude directly.

- **Installed:** `claude --version` → `2.1.287 (Claude Code)`.
- **CLAUDE.md loaded (the `/memory` check):** asked "Which CLAUDE.md files are loaded in this session?" Claude answered that exactly one is loaded, the project's root `CLAUDE.md`, and that there is no user-level, parent-directory, `.claude/CLAUDE.md` or `CLAUDE.local.md` file. Asked "How do I run the tests here?" with no other context, it answered `npm test`, `node --test tests/users.test.js` for one file, and that CI runs lint first, citing the Commands section of `CLAUDE.md`.
- **Rules loaded (the `/permissions` check):** asked Claude to list the rules in `.claude/settings.json`. It listed allow `npm test`, `npm run lint`, `node --test:*`; ask `git push:*`; deny `Read(./.env)`, `Edit(./.env)`, `git push --force:*`, `git push -f:*`, `git reset --hard:*`, and noted that deny wins over ask, so a force-push is blocked even though it also matches the `git push` ask rule.
- **Deny rule in action:** asked "Read the .env file and tell me what's in it." Claude refused, citing the `Read(./.env)` deny rule, and said it wouldn't get around it with `cat` or another shell command.
- **Ask rule in action:** asked Claude to "run git push". Before anything ran, Claude Code paused with "Needs your input" and showed the exact command (`git push -u origin set-up-claude`) with **Deny** / **Allow once** buttons, even though the session was in Auto mode. I clicked Deny, and Claude confirmed the push didn't run and didn't retry it.

One thing this taught me: `Read(./.env)` covers Claude's file-reading tool, not shell commands. Claude chose not to use `cat`, but the rule itself doesn't stop it, so a stricter setup would also deny `Bash(cat .env)`.
