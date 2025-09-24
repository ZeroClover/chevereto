# Chevereto Image Builder

<img alt="Chevereto" src="chevereto.svg" width="100%">

This repository houses the minimal assets required to build and publish Chevereto V4 Docker images. It targets the **pro** edition and expects a valid license key at build time.

## Repository Layout
- `Dockerfile` – runtime image definition based on `php:<version>-apache`, preloading the tooling needed by Chevereto.
- `scripts/chevereto/` – helper utilities copied into the image (filesystem sync, demo/import tooling).
- `.github/workflows/build-chevereto-image.yml` – GitHub Actions workflow that downloads the Chevereto package, tags the image with semantic versions, adds OCI metadata labels, and pushes to `ghcr.io`.
- `AGENTS.md` – contributor quickstart.

## GitHub Actions Workflow
The workflow runs nightly and offers manual dispatch:
1. Verifies the `CHEVERETO_LICENSE_KEY` secret.
2. Downloads the requested package version from `chevereto.com`.
3. Inspects GHCR for the latest patch tag and skips the build if already current (scheduled runs only).
4. Builds with `docker buildx`, applying tags `major`, `major.minor`, `major.minor.patch`, plus OCI labels (`org.opencontainers.image.version`, `…revision`, `…source`).
5. Pushes to the registry defined by `CHEVERETO_IMAGE_REGISTRY` (defaults to `ghcr.io/zeroclover/chevereto`).

To trigger manually, open the *Actions* tab → **Build Chevereto Image** → *Run workflow*, optionally setting the `version` input.

## Local Build (Advanced)
> Building locally still requires the pro package. Provide `CHEVERETO_LICENSE_KEY` before running.

```sh
export CHEVERETO_LICENSE_KEY=your_license_key
export VERSION=4.3.5
curl -fsSLJO -H "License: $CHEVERETO_LICENSE_KEY" "https://chevereto.com/api/download/$VERSION"
unzip -oq chevereto_*.zip -d chevereto
docker build --build-arg VERSION=$VERSION --build-arg PHP=8.2 -t chevereto:$VERSION .
rm -rf chevereto chevereto_*.zip
```

Apply the same tags the workflow generates (`$VERSION`, `${VERSION%.*}`, `${VERSION%%.*}`) before pushing to your registry of choice.

## Notes
- Never commit license keys or downloaded packages; the workflow cleans up artifacts automatically.
- Update `CHEVERETO_IMAGE_REGISTRY` or the workflow file if your organization publishes under a different namespace.
- The legacy deployment assets (compose templates, make targets, etc.) are intentionally removed to keep this repository focused on image production.
