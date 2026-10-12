# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

A small Express REST API (users + health check) backed by an in-memory store. It is a starter project for the Claude Code course.

## Commands

- `npm run dev` runs the API on http://localhost:3000 and restarts on file changes (`node --watch`). Set `PORT` to change the port.
- `npm test` runs every test with Node's built-in runner (`node --test`).
- `node --test tests/users.test.js` runs one file. Add `--test-name-pattern="404"` to run matching tests only.
- `npm run lint` runs ESLint (`eslint:recommended`). CI runs lint and then tests on Node 22, so both must pass.

## Conventions

- Use CommonJS (`require` / `module.exports`), not ES modules. ESLint is set to `sourceType: "script"`.
- Write tests with `node:test`, `node:assert`, and `supertest` against the exported `app`. Do not use Jest or Mocha, and do not start a real server in tests.
- Route handlers never touch data directly. All reads and writes go through functions exported from `db/store.js`; add a new store function when you need one.
- Error responses use the shape `{ error: "<message>" }` with the right status code (400 for bad input, 404 for not found, 500 for server errors). Successful creates return 201 and successful standard requests 200.
- Unused handler params named `req`, `res`, `next`, or `_` are allowed by the lint config. Other unused variables produce warnings.

## Architecture

- `server.js` builds the Express app, mounts each router under its path (`/users`, `/health`), and exports `app`. It only calls `listen()` when run directly (`require.main === module`), which lets tests import it.
- `routes/` has one file per resource, and each exports an `express.Router()`. A new resource needs a new file here plus an `app.use(...)` line in `server.js`.
- `db/store.js` holds data in module-level variables. Nothing persists across restarts, and state is shared between all tests in the same process: a POST in one test is visible to later tests.
