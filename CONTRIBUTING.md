# Contributing

## Setup

```bash
git clone https://github.com/stellarbrief/upgrade-preflight.git
cd upgrade-preflight
npm ci
```

Use Node 22, which is what CI uses (`package.json` requires 20.11 or newer). `npm ci` installs
exactly what `package-lock.json` says. `npm install` can rewrite that file, so do not commit
changes to `package-lock.json` unless you changed a dependency on purpose.

Docker is required for the integration test suite and for actually running the CLI, but not
for most day-to-day work (lint, typecheck, unit tests, and the build all run without it).

## Development loop

```bash
npm run lint          # eslint
npm run typecheck     # tsc --noEmit
npm run test          # unit tests (no Docker required)
npm run test:integration  # integration tests (Docker required; skip cleanly without it)
npm run build          # compile to dist/
```

## Branch / PR flow

1. Fork the repo and create a branch off `main`.
2. Make your change, with tests — see "What needs tests" below.
3. Run the full development loop above before opening a PR.
4. Open a PR using the template; link the issue you're closing, if any.

The PR template asks about `npm run test:integration`. Mark it N/A only when the change cannot
affect anything that runs against Docker: documentation, report formatting, config validation,
or changes to unit tests only, and say which in the pull request. If you change the behaviour of
`src/network/`, `src/runner/` or `src/sdk/`, run it, or say why you could not. CI runs the
integration job on every pull request either way.

## Picking up an issue

Comment on the issue to say you would like it, and wait for the maintainer to assign it to you
before you start. A comment alone does not reserve it. If an assigned issue has had no activity
for 7 days, the maintainer may ask whether you are still working on it, and may unassign it
after 7 more days without a reply. A pull request for an issue that is assigned to someone else
is looked at after theirs.

## Your first pull request

The first time you open a pull request, GitHub holds its CI run until a maintainer approves it,
so the checks show nothing for a while. That is a GitHub setting, not broken CI. The maintainer
approves the run when they review. Run the development loop locally in the meantime.

## AI-assisted contributions

AI-assisted contributions are welcome, as is this project's own use of AI assistance. You are
responsible for what you submit: you have run it, you understand it, and every claim in the
description is true. Pull requests are reviewed the same way whoever or whatever wrote them.

## Code style

TypeScript strict mode, ESM throughout (`"type": "module"`), relative imports use `.js`
extensions even though the source files are `.ts` — this is the standard pattern for
`moduleResolution: "NodeNext"` and lets `tsx` (dev) and `tsc` (build) both resolve the same
import correctly. ESLint (`typescript-eslint` recommended rules) enforces the rest.

## What needs tests

- **`src/diff/` and `src/report/`** are pure functions over plain data — every new verdict rule
  or report field needs a unit test here. This is the easiest, fastest place to add coverage
  and needs no Docker at all.
- **`src/config/`** — new schema fields need both an acceptance test and a rejection test.
- **`src/network/`** — mock `node:child_process` and `fetch`, following
  `src/network/quickstart.test.ts`. Assert the exact Docker flags, not just "it was called."
- **`src/sdk/` and `src/runner/`** — these touch a real network and are harder to unit-test in
  isolation; if you're adding a scenario type or arg type, a unit test on `src/sdk/args.ts`'s
  encoding is usually enough, with an integration test covering the real end-to-end path.

## How issues are rated for complexity

Issues in [`ISSUES_BACKLOG.md`](ISSUES_BACKLOG.md) are rated **Trivial**, **Medium**, or
**High** by scope and complexity:

- **Trivial**: typos, small bug fixes, a new example scenario, better error messages.
- **Medium**: a standard new feature or an involved bug fix — a new report format, a new arg
  type, CI improvements.
- **High**: a complex feature, refactor, or new integration — a new network backend, a plugin
  system, protocol-matrix (N-version) runs.

Every open issue carries `help wanted` and a `complexity:` label that matches its rating.
Trivial issues also carry `good first issue`; Medium and High do not.

## How maintainers work here

- There is currently one maintainer. Response times are best effort; there is no guaranteed
  turnaround.
- A bug report is reproduced before a fix is accepted. A feature is discussed in its issue
  before a PR is opened.
- CI (lint, typecheck, tests, build, and the integration job) must pass before merge.
- A change that affects a documented claim updates the docs in the same PR. A claim about
  behavior is accepted only with a test or a real run behind it.
- Changes to verdict rules, the config schema or the JSON report shape need maintainer approval,
  and a JSON shape change bumps `schemaVersion` and is noted in `CHANGELOG.md`.
- Releases are tags (`vX.Y.Z`) on `main` after CI is green, with notes taken from `CHANGELOG.md`.
- Security reports: see [`SECURITY.md`](SECURITY.md).
