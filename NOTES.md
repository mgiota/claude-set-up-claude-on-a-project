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

I ran these checks in a Claude Code on the web session (a cloud container), because my own OS can't run Claude Code locally.

- `claude --version` → `2.1.287 (Claude Code)`.
- In a fresh session started in the repo, I asked "How do I run the tests here?" without any other context. Claude answered `npm test` for the full suite, `node --test tests/users.test.js` for a single file, and that CI runs `npm run lint` first, and it named the Commands section of `CLAUDE.md` as its source. That confirms `CLAUDE.md` is loaded.
- The same session printed "Ignoring 3 permissions.allow entries from .claude/settings.json: this workspace has not been trusted", so it does read `.claude/settings.json` and finds my three allow rules. Claude Code holds back allow rules until a workspace is trusted, while deny and ask rules apply regardless, which is the safe default.
- `/memory` and `/permissions` are interactive-only commands and reported "isn't available in this environment" in the non-interactive session, so I couldn't view those screens directly. The two checks above cover the same ground: one shows `CLAUDE.md` loaded, the other shows `settings.json` loaded.
