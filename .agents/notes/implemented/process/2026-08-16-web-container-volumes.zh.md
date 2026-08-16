# Agent Note: Web 容器必须挂载的 volume

Status: implemented

[English](2026-08-16-web-container-volumes.md) | 中文

## 问题

不持久化 harness home 的 `dsh web` 容器在重建时会丢失 sessions、settings 和凭据。不 bind 宿主机项目的容器会让 agent（智能体）面对空的 cwd。写进 `$DSH_HOME/cordis.patch.yml` 的监听 overlay 会在该 home 成为 volume 时消失，已发布的端口随之失效。

## 决策

[`apps/web/Dockerfile`](../../../../apps/web/Dockerfile) 与 [`apps/web/compose.yml`](../../../../apps/web/compose.yml) 中的 Web UI 镜像要求两处挂载，说明见 [`apps/web/README.md`](../../../../apps/web/README.md)：

- `/workspace` 是 `$DSH_WORKSPACE`（宿主机绝对路径）的 bind。它是进程 cwd 和默认工作区根目录。
- `/home/node/.dsh` 是命名 volume `dsh-home`。它是 harness home：`sessions/`、`settings.yaml`、`.credentials.yaml`、`storages/`、`profiles/web`。

全接口监听 overlay 为 `/etc/dsh/docker.cordis.patch.yml`，通过 `dsh web --patch` 应用。它不在 harness home 下，因此 home volume 不会替换它。未设置 `DSH_WORKSPACE` 时 Compose 失败。

## 曾考虑的替代方案

**用 home 级 `cordis.patch.yml` 承载监听 overlay。** 不予采纳：必需的 `/home/node/.dsh` volume 会盖住镜像里的文件，于是 `docker run -p`／Compose 端口发布只会绑定回环地址。

**只使用一个 volume。** 不予采纳：项目文件与 harness home 的生命周期和所有权不同。混在一处会把凭据放进项目旁，或把 session 数据留在工作区树里。

**带默认值的可选挂载。** 不予采纳：未设置的工作区 bind 看起来像 UI 已运行，但 agent 看不到宿主机项目；Compose 的 `:?` 检查会在启动时让这种情况失败。

## 后果

运维必须设置 `DSH_WORKSPACE` 并保留两处挂载。覆盖镜像 command 时必须保留 `--patch /etc/dsh/docker.cordis.patch.yml`，否则已发布端口会绑定回环地址。进程 uid 为 `1000`；工作区 bind 必须对该 uid 可写。
