# `@deepseek-ai/dsh-web-frontend`

[English](README.md) | 中文

用于构建 `dsh web` 所服务的浏览器 shell 的 Vite 入口。本目录同时提供 Web UI 容器（[Dockerfile](Dockerfile)、[compose.yml](compose.yml)），其中安装已发布的 `@deepseek-ai/dsh@0.1.0-rc.6`。

## 运行容器

镜像版本为 `0.1.0-rc.6`。在本目录执行：

```sh
export DSH_WORKSPACE=/absolute/path/to/your/project
docker compose up --build
```

打开 `http://127.0.0.1:3080`，然后把 `/workspace` 选为工作区。可在环境中传入 `DEEPSEEK_API_KEY`，或等 UI 启动后在 **Settings → Models** 中保存密钥。详见 [Web UI 指南](../../docs/user/guide/index.md)。

## 必须挂载的 volume

两处挂载均为必需。省略 `/workspace` 时，agent（智能体）看不到宿主机项目。省略 `/home/node/.dsh` 时，重建容器会丢失 sessions、settings 和凭据。

| 容器路径 | Compose 来源 | 存放内容 |
|---|---|---|
| `/workspace` | `$DSH_WORKSPACE` 的 bind | 进程 cwd 与默认工作区根目录。agent 在此读写文件。未设置 `DSH_WORKSPACE` 时 Compose 失败。 |
| `/home/node/.dsh` | 命名 volume `dsh-home` | Harness home：`sessions/`、`settings.yaml`、`.credentials.yaml`、`storages/`、`profiles/web`。 |

监听 overlay 位于镜像内的 `/etc/dsh/docker.cordis.patch.yml`，通过 `--patch` 应用。不要挂载覆盖 `/etc/dsh`。挂载 `/home/node/.dsh` 不会替换该 overlay。

进程以 uid `1000`（`node`）运行。`$DSH_WORKSPACE` 必须对该 uid 可写。

在本目录运行 `docker compose` 会使用 [compose.yml](compose.yml)。等价的 `docker run`：

```sh
docker build -t dsh-web:0.1.0-rc.6 -f Dockerfile .
docker run --rm -p 3080:3080 \
  -e DEEPSEEK_API_KEY \
  -v "$DSH_WORKSPACE":/workspace \
  -v dsh-home:/home/node/.dsh \
  dsh-web:0.1.0-rc.6
```
