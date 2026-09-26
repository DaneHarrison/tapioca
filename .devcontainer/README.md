# Developing tapioca in a Dev Container

This gives you a Linux environment with the exact Ruby version this repo
requires (`.ruby-version`) and all native build tooling preinstalled, so you
don't have to manage Ruby versions or compile native gem extensions
(`google-protobuf`, `nokogiri`, `sqlite3`, `bcrypt`, etc.) on the host.

## Opening it

1. Clone your fork of [Shopify/tapioca](https://github.com/Shopify/tapioca).
2. Open the folder in VS Code.
3. Run **Dev Containers: Reopen in Container** from the command palette
   (or click the prompt VS Code shows automatically).
4. First build installs apt packages and runs `bin/setup` (`bundle install`)
   — takes a few minutes. Later reopens are fast, since gems are cached in a
   named Docker volume across rebuilds.

## Everyday loop

- Edit files normally in the editor. The workspace is bind-mounted into the
  container, so changes are visible to the container instantly — no watch
  or sync step needed.
- Use the VS Code integrated terminal (already running inside the
  container) to run commands. These mirror what CI's `linters` job runs
  (`.github/workflows/ci.yml`):
  - `bin/test` — run the full test suite (or `bin/test spec/path/to/foo_spec.rb`
    for a single file). See [`test.md`](test.md) for subsets and parallel runs.
  - `bin/style` — Rubocop.
  - `bin/typecheck` — Sorbet type check.
  - `bin/docs` / `bin/readme` — documentation verification.
  - `bundle exec exe/tapioca gem --verify` — check gem RBIs are up to date.
  - `bundle exec exe/tapioca check-shims` — check for duplicate shims.

## Branch layout

- `main` mirrors [Shopify/tapioca](https://github.com/Shopify/tapioca) `main`
  exactly. Feature/PR branches start from here.
- `docker-main` is `main` plus this Docker setup (`.devcontainer/` and
  `.gitattributes`). It's the only branch that tracks these files.

On `main` and feature branches, the Docker files are local, untracked files
that git ignores, so they never end up in a commit or a PR.

### One-time setup per clone

```bash
git remote add upstream https://github.com/Shopify/tapioca.git
git remote set-url --push upstream NO_PUSH_TO_UPSTREAM   # can't push to Shopify by accident
printf '.devcontainer/\n.gitattributes\n' >> .git/info/exclude
git config alias.docker-on '!git restore --source=origin/docker-main --worktree -- .devcontainer .gitattributes'
git checkout main && git docker-on
```

### Everyday commands

| Task | Command |
|---|---|
| Sync `main` with Tapioca | `git checkout main && git pull upstream main && git push origin main` |
| Start a feature | `git checkout -b fix/xyz main` |
| Docker files missing | `git docker-on` |
| Rebase `docker-main` onto new `main` | `git checkout docker-main && git rebase main && git push --force-with-lease` |
| Change the Docker setup | `git checkout docker-main`, edit, commit (`git add -f`, because of the exclude rule), push, `git checkout <branch>`, `git docker-on` |

## Contribution workflow

1. Branch off `main` in your fork.
2. Make your change, with tests.
3. Run the checks above locally before pushing — they're the same ones CI
   runs, so passing them locally means a green PR.
4. Open a PR against `Shopify/tapioca` using the repo's PR template
   (`.github/pull_request_template.md`): Motivation, Implementation, Tests.
   Reference the issue number you're fixing (e.g. "Fixes #2082").
5. If this is your first contribution, the Shopify CLA bot will comment on
   your PR with a link to sign the CLA. After signing, comment `signed` on
   the PR to re-trigger the check.

See the main [`README.md`](../README.md)'s "Contributing" and "DSL
compilers" sections for project-specific conventions (e.g. how DSL
compilers are structured) before diving into a fix.

## Rebuilding the container

If `.devcontainer/Dockerfile` changes (e.g. a new apt package is needed),
run **Dev Containers: Rebuild Container** from the command palette.
