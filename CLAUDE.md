# CLAUDE.md

Small Express REST API (users + health check) backed by an in-memory store; it's the starter project for the Claude Code course.

## Commands

- `npm install` — install dependencies
- `npm run dev` — start the API with auto-restart on http://localhost:3000 (`PORT` overrides)
- `npm test` — run all tests (Node's built-in test runner)
- `node --test tests/users.test.js` — run a single test file
- `npm run lint` — ESLint; CI runs `npm run lint` then `npm test` on Node 22, so both must pass

## Architecture

- `server.js` builds the app, mounts each router under `/<resource>`, and exports `app`. It only calls `listen` when run directly, so tests import `app` without opening a port.
- `routes/` holds one file per resource, each exporting an `express.Router()`. A new resource gets its own file plus one `app.use("/<resource>", ...)` line in `server.js`.
- `db/store.js` is the only module that touches data. It's in memory and resets on restart; there is no real database.

## Conventions

- Use CommonJS (`require` / `module.exports`), not `import`/`export`: ESLint parses files as scripts and will fail on ES modules.
- Routes read and write data only through functions exported by `db/store.js`; to add data access, add a function there instead of editing the `users` array from a route.
- Return errors as `res.status(<code>).json({ error: "<message>" })` and `return` immediately: 400 for missing or invalid input, 404 when a record doesn't exist.
- Write tests with `node:test`, `node:assert` and `supertest` against the exported `app`, not Jest or Mocha. The store is shared across tests in a run, so don't assert exact user counts or IDs created by other tests.
- Don't change `package.json` dependencies without asking first.
