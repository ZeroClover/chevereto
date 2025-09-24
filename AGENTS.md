# Repository Guidelines

## Project Structure & Module Organization
- Root `Makefile` orchestrates image builds, namespace provisioning, and utility targets; environment comes from `.env` plus files under `namespace/`.
- Compose templates (`default.yml`, `dev.yml`, `docker-compose.yml.dist`) define production and local stacks; adjust copies rather than editing templates in place.
- Automation scripts live in `scripts/system` (provisioning, deploy, updates) and `scripts/chevereto` (dev file sync/watch). Reference `docs/` for task guides (e.g., `docs/DEV.md`, `docs/COMMANDS.md`).

## Build, Test, and Development Commands
- `make env` – prompts for licensing and DNS values, writes `.env` once.
- `make setup` – provisions cron + nginx-proxy/acme companion; run after `make env` on fresh hosts.
- `make image` / `make image-custom TARGET=dev PHP=8.2` – build Chevereto images (paid or dev flavor).
- `make deploy NAMESPACE=demo ADMIN_EMAIL=admin@example.com` – creates namespace file, optional Cloudflare DNS, boots stack, and runs app installer.
- `make up-d NAMESPACE=demo` / `make down NAMESPACE=demo` – start/stop compose stack; use `make re-up-d` after config changes.
- `make log` / `make exec NAMESPACE=demo COMMAND="app/bin/cli -C status"` – tail container logs or run Chevereto CLI.

## Coding Style & Naming Conventions
- Bash scripts begin with `#!/usr/bin/env bash`, `set -e`, uppercase env vars, and `make --no-print-directory`; keep spacing and quoting consistent.
- Compose YAML relies on `${VAR}` substitutions populated by `.env` and namespace files; comment toggles (e.g., `CHEVERETO_SERVICING`) should remain.
- Docs use Markdown with concise headings and fenced `sh` snippets mirroring README tone.

## Testing Guidelines
- No automated test suite; validate changes by reloading the target namespace (`make re-up-d`) and smoke-testing the Chevereto UI.
- Monitor `make log` and database/redis health after updates; confirm `app/bin/cli -C update` completes when bumping images.

## Commit & Pull Request Guidelines
- Write focused commits describing the operation (e.g., "makefile: tweak namespace feedback"); reference namespaces or scripts touched.
- Pull requests should outline intent, list impacted commands or services, and note any manual verification (logs watched, UI checked). Include screenshots when altering nginx or app-facing behavior.

## Security & Configuration Tips
- Never commit `.env`, namespace secrets, or built images. Use `make env` interactively per host.
- When enabling HTTPS or Cloudflare automation, confirm certificates and DNS tokens via `make proxy --view` and retain minimal permissions on API keys.
