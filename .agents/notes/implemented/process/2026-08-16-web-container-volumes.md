# Agent Note: Web container required volume mounts

Status: implemented

English | [中文](2026-08-16-web-container-volumes.zh.md)

## Problem

A `dsh web` container that does not persist harness home loses sessions, settings, and credentials on recreate. A container that does not bind a host project leaves the agent with an empty cwd. A listen overlay written into `$DSH_HOME/cordis.patch.yml` disappears when that home is a volume, so published ports stop working.

## Decision

The Web UI image in [`apps/web/Dockerfile`](../../../../apps/web/Dockerfile) and [`apps/web/compose.yml`](../../../../apps/web/compose.yml) requires two mounts, documented in [`apps/web/README.md`](../../../../apps/web/README.md):

- `/workspace` is a bind of `$DSH_WORKSPACE` (absolute host path). It is process cwd and the default workspace root.
- `/home/node/.dsh` is named volume `dsh-home`. It is harness home: `sessions/`, `settings.yaml`, `.credentials.yaml`, `storages/`, `profiles/web`.

The all-interfaces listen overlay is `/etc/dsh/docker.cordis.patch.yml`, applied with `dsh web --patch`. It is not under harness home, so the home volume does not replace it. Compose fails if `DSH_WORKSPACE` is unset.

## Alternatives considered

**Home-level `cordis.patch.yml` for the listen overlay.** Rejected: a required `/home/node/.dsh` volume hides the image's file, so `docker run -p` / Compose port publishing would bind loopback only.

**A single volume.** Rejected: project files and harness home have different lifetimes and ownership. Mixing them would store credentials next to the project or keep session data in the workspace tree.

**Optional mounts with defaults.** Rejected: an unset workspace bind looks like a running UI while the agent cannot see the host project; Compose's `:?` check fails that case at start.

## Consequences

Operators must set `DSH_WORKSPACE` and keep both mounts. Overriding the image command must keep `--patch /etc/dsh/docker.cordis.patch.yml` or published ports bind loopback. The process uid is `1000`; the workspace bind must be writable by that uid.
