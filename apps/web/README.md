# `@deepseek-ai/dsh-web-frontend`

English | [中文](README.zh.md)

Vite entry that builds the browser shell served by `dsh web`. This directory also ships the Web UI container ([Dockerfile](Dockerfile), [compose.yml](compose.yml)), which installs published `@deepseek-ai/dsh@0.1.0-rc.6`.

## Run the container

The image version is `0.1.0-rc.6`. From this directory:

```sh
export DSH_WORKSPACE=/absolute/path/to/your/project
docker compose up --build
```

Open `http://127.0.0.1:3080`, then choose `/workspace` as the workspace. Pass `DEEPSEEK_API_KEY` in the environment, or save a key in **Settings → Models** after the UI starts. See the [Web UI guide](../../docs/user/guide/index.md).

## Required volume mounts

Both mounts are required. Omitting `/workspace` leaves the agent without the host project. Omitting `/home/node/.dsh` drops sessions, settings, and credentials when the container is recreated.

| Container path | Compose source | What it holds |
|---|---|---|
| `/workspace` | bind of `$DSH_WORKSPACE` | Process cwd and default workspace root. The agent reads and edits these files. Compose fails if `DSH_WORKSPACE` is unset. |
| `/home/node/.dsh` | named volume `dsh-home` | Harness home: `sessions/`, `settings.yaml`, `.credentials.yaml`, `storages/`, `profiles/web`. |

The listen overlay is `/etc/dsh/docker.cordis.patch.yml` inside the image, applied with `--patch`. Do not mount over `/etc/dsh`. A volume on `/home/node/.dsh` does not replace that overlay.

The process runs as uid `1000` (`node`). `$DSH_WORKSPACE` must be writable by that uid.

`docker compose` from this directory uses [compose.yml](compose.yml). Equivalent `docker run`:

```sh
docker build -t dsh-web:0.1.0-rc.6 -f Dockerfile .
docker run --rm -p 3080:3080 \
  -e DEEPSEEK_API_KEY \
  -v "$DSH_WORKSPACE":/workspace \
  -v dsh-home:/home/node/.dsh \
  dsh-web:0.1.0-rc.6
```
