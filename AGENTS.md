# Repository guidance

## Purpose

This repository is a deliberately small local-development template for a Node.js TypeScript CLI or bot with
optional MySQL. Preserve that simplicity: avoid frameworks, extra services, build layers, or abstractions unless the
requested feature genuinely requires them.

This is not a production deployment template. The Compose database uses an empty root password, exposes port 3306,
and disables durability features for faster local development.

## Repository map

- `script/src/` contains the ESM application, npm manifest and lockfile, `.nvmrc`, and TypeScript configuration.
- `script/docker.dev/` contains the development image, dependency bootstrap, supervisord, and nodemon setup.
- `database/etc/` contains the optional MySQL development configuration.
- `compose.yml` connects the script and database services for local development.
- `.circleci/config.yml` installs from the lockfile, type-checks, and smoke-runs the script.
- `renovate.json` owns automated dependency and runtime updates.

The Docker build context is `script/`; paths used by `script/docker.dev/Dockerfile` are relative to that directory.

## Keep the project simple

- Keep TypeScript running directly through `tsx`; `tsc --noEmit` is validation, not a build pipeline.
- Preserve ESM and strict TypeScript unless a requested change needs a different runtime model.
- Prefer stable npm scripts and shared configuration over one-off commands or CI-only hotfixes.
- Add tests, tooling, or services only when they protect real behavior introduced by the task.
- Keep changes focused. Do not mix unrelated dependency upgrades or broad cleanup into a targeted change.

## Node runtime invariant

Use only an exact, bare `satantime/puppeteer-node:X.Y.Z` image tag.

- Do not use the official `node` image.
- Do not use floating aliases such as `latest`, `lts`, `current`, or an LTS codename.
- Do not use partial tags or distro suffixes such as `bookworm` or `alpine`.
- Choose the newest stable Node LTS version for which that exact `satantime/puppeteer-node` tag exists. If upstream
  Node is newer but the matching image has not been published, retain the newest available image-backed LTS version.

Keep the same exact runtime version synchronized in all five locations:

- `script/docker.dev/Dockerfile`
- `.circleci/config.yml`
- `script/src/.nvmrc`
- `script/src/package.json` under `engines.node`
- `script/src/package-lock.json` under `packages[""].engines.node`

Update these references together. Regenerate the lockfile through npm after changing `package.json`; do not hand-edit
transitive lockfile entries.

The existing Renovate rules intentionally map native Node version references to published
`satantime/puppeteer-node` tags, apply Node LTS versioning, and group all runtime references into one update. Keep that
native-manager approach instead of adding even-major heuristics, duplicate regex managers, or file-specific hotfixes.
If another Node runtime reference is added, ensure it participates in the same Renovate update.

`@types/node` belongs in the same Renovate PR, but it is independently published. Its exact minor and patch numbers do
not need to equal the Node runtime version.

## Dependencies and generated state

- Run npm commands against `script/src/` and keep dependency versions exactly pinned.
- Commit `script/src/package-lock.json` whenever `script/src/package.json` changes.
- Use `npm ci` to verify the committed lockfile.
- Leave routine dependency upgrades to Renovate. Do not run `npm audit fix`, force upgrades, or change security
  automation unless the task explicitly requests it.
- Do not commit `.env`, `database/src/`, `node_modules/`, `dist/`, or other generated runtime state.
- Never commit registry credentials or tokens. Public image lookups need no Docker Hub credentials; use external
  repository or service secrets when authentication is actually required.

## Validation

Run the smallest relevant set, from the repository root:

```sh
npm --prefix script/src ci
npm --prefix script/src run typecheck
npm --prefix script/src start
docker compose config --quiet
```

For Docker, entrypoint, runtime, or Compose changes, also run:

```sh
docker compose build script
```

Use `docker compose up --build` when the change needs the full script-plus-MySQL integration path. It is not required
for documentation-only or otherwise unrelated changes. Test container lifecycle changes inside the image because the
entrypoint and watcher depend on Linux utilities and container paths.

When `renovate.json` changes, validate it with a current Renovate release and run a local lookup dry-run. Check the
semantic result, not only the exit code: Node runtime references must resolve to one exact image-backed LTS version and
one `renovate/root/node` branch.

Run `bash -n` on each changed shell script.

## Style

Follow `.editorconfig`: UTF-8, LF line endings, two-space indentation, final newlines, trimmed trailing whitespace, and
a 120-character line limit. Preserve unrelated working-tree changes.
