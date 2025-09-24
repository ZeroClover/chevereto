# Repository Guidelines

## Project Structure & Module Organization
The repository carries only the assets needed to build Chevereto V4 Pro Docker images. `Dockerfile` defines the php-apache runtime, while `scripts/chevereto/` holds the rsync and inotify helpers that run inside the container. The GitHub Actions workflow in `.github/workflows/build-chevereto-image.yml` drives nightly and manual builds, and you are expected to drop the unpacked Chevereto release into `./chevereto/` before any local image build.

## Build, Test, and Development Commands
- `export CHEVERETO_LICENSE_KEY=…` and `export VERSION=4.x.y` — make the required license and version available to the build context.
- `curl -fsSLJO -H "License: $CHEVERETO_LICENSE_KEY" "https://chevereto.com/api/download/$VERSION"` — download the matching Pro bundle; unzip it into `chevereto/`.
- `docker build --build-arg VERSION=$VERSION --build-arg PHP=8.2 -t chevereto:$VERSION .` — reproduce the production image locally.
- `docker run --rm -p 8080:80 chevereto:$VERSION` — smoke-test the container before submitting changes.
- `gh workflow run build-chevereto-image.yml --ref main -f version=$VERSION` — trigger the publishing workflow without waiting for the nightly schedule.

## Coding Style & Naming Conventions
Keep Bash scripts executable, start them with `#!/usr/bin/env bash`, and retain `set -e` for fail-fast behaviour. Use lowercase snake case for shell functions and upper snake case for environment variables (for example, `CHEVERETO_SESSION_SAVE_PATH`). The Dockerfile aligns multiline instructions with four-space indents and groups related packages; match that layout when adding dependencies or args.

## Testing Guidelines
There is no formal test suite, so rely on Docker-based verification. Build locally after any change to the Dockerfile or scripts, run the container against a temporary volume, and confirm that syncing utilities in `scripts/chevereto/` still move files as expected. If you touch Bash logic, run `shellcheck scripts/chevereto/*.sh` to catch obvious regressions.

## Commit & Pull Request Guidelines
History is sparse, so favour concise, imperative commit summaries such as `Update sync script for vendor caching`. Reference related issues in the body, mention any required env var changes, and attach logs or `docker build` output when relevant. Pull requests should explain build impact, note if GHCR tags or registry defaults change, and include screenshots only when UI assets are affected.

## Security & Configuration Tips
Never commit downloaded Chevereto archives or license keys; the workflow retrieves archives securely using repository secrets. Ensure `CHEVERETO_LICENSE_KEY` and `CHEVERETO_IMAGE_REGISTRY` remain configured in repository settings, and scrub generated assets during cleanup. When sharing debugging snippets, redact URLs or credentials embedded in environment variables.
