# AGENTS.md

## About the project
docker-rockylinux-ansible is a Rocky Linux image with Ansible pre-installed,
designed for testing playbooks and roles. All versions live in a single repository
and are published under different tags to Docker Hub and GHCR.

## Repository structure
- `rocky<version>/` — separate build context: `Dockerfile`, `.dockerignore`.
- `.github/workflows/`:
  - `ci.yml` — checks on PR/push to master; calls reusable `test-image.yml`.
  - `build.yml` — manual build check across all versions and platforms
    (`workflow_dispatch`); calls reusable `build-image.yml` with `push: false`.
  - `release.yml` — publishing on tags: `validate-tag` → `config` → `test` → matrix
    `release` (reusable `build-image.yml`, `secrets: inherit`).
  - `config.yml` — reusable workflow, single source of constants: outputs
    `versions` (JSON array), `platforms` (JSON array),
    `version_platforms` (JSON object mapping a version to its platforms),
    `build_matrix` (JSON array of version/platform pairs), `latest_version`
    (string); callers drive their matrices from `needs.config.outputs.*`.
  - `test-image.yml`, `build-image.yml` — reusable workflows with shared steps.
- `Makefile` + `scripts/test.mk` — `make test-build` smoke test mirroring
  `test-image.yml`; the version is set via the `version_branch` variable (default:
  the latest supported version, `10.0`).
- `.context/optimize-ci.md` — CI/CD architecture plan and rationale (read before
  changing the workflows).
- `.gitignore` — files kept out of git (`.context`, `.omo`).

## Build and test
- Local build: `cd rocky10 && docker build -t docker-rockylinux-ansible . --build-arg VERSION=10.0`
- Smoke test, as in CI (`test-image.yml`):
  1. `podman build -t docker-rockylinux-ansible:<version> ./rocky<major>/. --build-arg VERSION=<version>`
  2. `podman run --name test-container --detach --privileged \
     --volume /sys/fs/cgroup:/sys/fs/cgroup:ro docker-rockylinux-ansible:<version>`
  3. `podman exec --tty test-container env TERM=xterm ansible --version`
- Or via the Makefile: `make test-build` (override the version with
  `make test-build version_branch=9.7`).

## CI/CD conventions
- Version matrix (image tags): `9.0`–`9.8`, `10.0`.
- Each version maps to a `rocky<major-version>` directory; the image tag is the
  full version; `10.0` is the latest and also carries the `latest` tag.
- `8.x` is archived — no longer built or published (see README, "Archive tags").
- Platforms per version:
  - `9.0`: `linux/amd64`, `linux/arm64/v8`.
  - `9.1`, `9.3`: also `linux/ppc64le`, `linux/s390x`.
  - Other `9.x`: also `linux/s390x`.
  - `10.0`: `linux/amd64`, `linux/arm64/v8`, `linux/ppc64le`, `linux/s390x`.
- Versions, platform mapping, and the `latest` tag mapping live in `config.yml`
  (outputs `versions`, `platforms`, `version_platforms`, `build_matrix`,
  `latest_version`) — edit it to change the build/test matrix; no other workflow
  needs editing for a version change.
- Release tag: `x.x` or `x.x.x`, optional `v` prefix
  (regex `^v?[0-9]{1,2}\.[0-9]{1,2}(\.[0-9]{1,2})?$`), no suffixes — enforced by
  the `validate-tag` job.
- `GITHUB_TOKEN` permissions are minimal: workflow-level `contents: read`.
  Registry-capable callers grant `packages: write` on their calling job
  (`release` in `release.yml`, `test-build` in `build.yml`), and `build-image.yml`
  requests it on the `build` job. Note: a called workflow's token is the
  *intersection* of the caller job's and the called job's permissions.
- Images are published to Docker Hub (`kirillzak/docker-rockylinux-ansible`) and
  GHCR (`ghcr.io/<owner>/<repo>`).

## Workflow and commit conventions
- Comments, commit messages, and pull requests are written in English.
- Work on a dedicated branch for a single issue; implement steps sequentially and
  commit after confirmation.
- A commit subject states only what was done (no plans or follow-up tasks) and is
  no longer than 70 characters.
- Conventional Commits types are used when they fit: `feat:`, `fix:`, `docs:`,
  `refactor:`, `chore:`, `ci:`, `test:`.

## How to add a new Rocky version
1. Create `rocky<version>/` with `Dockerfile` and `.dockerignore`.
2. Add `"<version>"` to the version table in `.github/workflows/config.yml` (and
   update `latest_version` there if the new version becomes `latest`); add the tag
   to README.
3. If the new version is the latest, update the `version_branch` default in
   `scripts/test.mk`.
4. Run `workflow_dispatch` on `build.yml` and run the local smoke test
   (`make test-build`).