# AGENTS.md

Guidance for AI agents working in this repository.

## Project

`totlog` is a minimal logger: categories, colors, and a `message` event that lets you plug in
appenders. It is deliberately (almost) dependency-free — `lodash` is the only runtime dependency, and
`colors` is optional. Do not add more without asking first.

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

This branch has no CI. Its `.travis.yml` was removed: travis-ci.org has been shut down for years, so
the file described a pipeline that could never run, and `master`'s GitHub Actions workflow is scoped
to `master`. Run the linter and the suite locally before committing.

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
