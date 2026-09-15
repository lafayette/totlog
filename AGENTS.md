# AGENTS.md

Guidance for AI agents working in this repository.

## Project

`totlog` is a minimal logger: categories, colors, and a `message` event that lets you plug in
appenders. It is deliberately (almost) dependency-free — do not add runtime dependencies without
asking first.

## Branches

- `master` — main line, targets current Node.js.
- `legacy` — maintenance line for **Node.js 8**. This is the branch you are on.

Keep the two branches feature-equivalent: when you add or change an appender on one, port it to the
other, adapting only syntax, tooling and formatting.

## This branch: Node.js 8

- ES2017 at most: no object spread, no optional chaining, no nullish coalescing.
- The toolchain is pinned to old majors on purpose: ESLint 3 (`.eslintrc.js`), Mocha 5, Istanbul,
  `mock-require` 2.
- Develop and test with Node 8 (`nvm use 8`), not with whatever Node is on `PATH`.

## Layout

| File | Purpose |
| --- | --- |
| `index.js` | logger factory, console output, `message` event emitter |
| `appenders.js` | built-in appenders: `slack`, `telegram`, `mattermost`, `logstash` |
| `stream.js` | `Writable` stream forwarding chunks to a logger |
| `tests/` | Mocha specs, one file per source file |

## Commands

```bash
nvm use 8
npm install
npm run lint
npm test
```

Known issue: the two `logstash` specs in `tests/appenders.js` fail on this branch — they were never
updated after logstash messages gained a trailing newline. Unrelated to any new work.

## Code style

Enforced by ESLint — run the linter instead of guessing.

- Tabs for indentation, no semicolons, single quotes, trailing commas in multiline literals.
- Space before function parens: `function slack (options) {`.
- `max-len` 120, `max-lines` 240 per file.
- `function` declarations for exported units, arrows for callbacks.

## Appender conventions

An appender is a factory that validates its options and returns `function (ev)` handling a single log
event (`{ time, level, category, message, content }`).

- Validate eagerly in the factory and throw `new Error('X is required.')` for missing options.
- Use the core `http`, `https`, `net` and `dgram` modules directly — no HTTP client libraries.
- Never throw from the returned handler. Report failures through the module's own silent logger
  (`log.error(...)`) so that logging can never break the host application.
- Truncate chat messages with `MAX_MESSAGE_LENGTH` (default 700), the way `telegram` and
  `mattermost` do.
- Export it from `module.exports` in `appenders.js` and cover it in `tests/appenders.js` by mocking
  `https` with `mock-require`.

## Git rules

- Local commits and `git fetch` are fine.
- **Never change anything on a remote without the user's explicit consent for that specific action.**
  This covers `git push` (including `--force`), creating or deleting remote branches and tags,
  opening or merging pull requests, and publishing to npm. Ask every time — consent for one action is
  not consent for the next.
