# AGENTS.md

Guidance for AI agents working in this repository.

## Project

`totlog` is a minimal logger: categories, colors, and a `message` event that lets you plug in
appenders. It is deliberately (almost) dependency-free — `lodash` is the only runtime dependency, and
`colors` is optional. Do not add more without asking first.

## Branches

- `master` — main line, targets current Node.js. This is the branch you are on.
- `legacy` — maintenance line for Node.js 8.

Keep the two branches feature-equivalent: when you add or change an appender on one, port it to the
other, adapting only syntax, tooling and formatting.

## This branch: current Node.js

- Modern syntax is fine, but stay inside the Node core API — no new dependencies.
- Toolchain: ESLint 9 flat config (`eslint.config.mjs`), Mocha 10, `nyc`, `mock-require` 3.
- CI lives in `.github/workflows/main.yml` (lint annotations, tests, coverage to Coveralls). It
  pins Node 24 on purpose: `node-version: latest` drifted onto Node 26, where mocha 10's bundled
  yargs fails to load, so the test step produced no report and the job died on the CTRF step
  instead of on the real error. Do not put `latest` back.

## Layout

| File | Purpose |
| --- | --- |
| `index.js` | logger factory, console output, `message` event emitter |
| `appenders.js` | built-in appenders: `slack`, `telegram`, `mattermost`, `logstash` |
| `stream.js` | `Writable` stream forwarding chunks to a logger |
| `tests/` | Mocha specs, one file per source file |

## Commands

```bash
npm install
npm run lint
npm run lint-fix
npm test
npm run test-coverage
```

## Code style

Enforced by ESLint (`@stylistic`) — run the linter instead of guessing.

- Two spaces for indentation, no semicolons, single quotes, trailing commas in multiline literals.
- Space before function parens: `function slack (options) {`.
- `function` declarations for exported units, arrows for callbacks.

## Appender conventions

An appender is a factory that validates its options and returns `function (ev)` handling a single log
event (`{ time, level, category, message, content }`).

- Validate eagerly in the factory and throw `new Error('X is required.')` for missing options.
- Use the core `http`, `https`, `net` and `dgram` modules directly — no HTTP client libraries.
- Never throw from the returned handler. Report failures through the module's own silent logger
  (`log.error(...)`) so that logging can never break the host application.
- Drain the response (`response.resume()`), otherwise the socket is never released and a short-lived
  process will not exit.
- Truncate chat messages with `MAX_MESSAGE_LENGTH` (default 700), the way `telegram` and
  `mattermost` do.
- Export it from `module.exports` in `appenders.js` and cover it in `tests/appenders.js` by mocking
  `https` with `mock-require`.

## Git rules

Local commits and `git fetch` are fine. Everything below is about writing to a remote.

- **Writing to a remote is strictly forbidden without the user's explicit consent for that specific
  operation.** This covers `git push` (including `--force`), creating or deleting remote branches and
  tags, opening or merging pull requests, and publishing to npm.
- Consent is scoped to the commits that already existed when it was given, and it expires with the
  message that gave it. Anything committed afterwards needs a new ask — a minute later, and even when
  it directly continues the work that was just approved. "You can push X" is never standing
  permission, and finishing a follow-up task does not re-authorize a push.
- A bare "yes" arriving together with a new task answers the task. It is not consent to push.
- Ask in the same turn you intend to push, immediately before pushing, and name exactly what will
  change: the remote, each refspec, and the old and new SHA. Then wait for the answer in the next
  message. Never push first and report afterwards.
- If you are unsure whether the consent you have covers what you are about to push, it does not.
  Ask again; the cost of asking is a sentence, the cost of being wrong is a force-push.
