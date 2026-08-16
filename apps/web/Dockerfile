# DeepSeek Harness Web UI. The image version is the published `@deepseek-ai/dsh`
# version; `docker run --rm <image> --version` must print the same value.
#
#   docker build -t dsh-web:0.1.0-rc.6 -f apps/web/Dockerfile .
#   docker run --rm dsh-web:0.1.0-rc.6 --version
#   docker run --rm -p 3080:3080 -e DEEPSEEK_API_KEY dsh-web:0.1.0-rc.6
FROM node:24-bookworm-slim AS install

ARG DSH_VERSION=0.1.0-rc.6

RUN apt-get update \
    && apt-get install -y --no-install-recommends python3 make g++ git ca-certificates \
    && rm -rf /var/lib/apt/lists/*

# Native addons (node-pty, koffi, spawn-helper) ship install scripts; keep them on.
RUN npm install -g --ignore-scripts=false "@deepseek-ai/dsh@${DSH_VERSION}" \
    && test "$(dsh --version)" = "${DSH_VERSION}"

FROM node:24-bookworm-slim

ARG DSH_VERSION=0.1.0-rc.6

LABEL org.opencontainers.image.title="dsh-web" \
      org.opencontainers.image.description="DeepSeek Harness Web UI (dsh web)" \
      org.opencontainers.image.version="${DSH_VERSION}" \
      org.opencontainers.image.source="https://github.com/deepseek-ai/deepseek-harness"

ENV DSH_VERSION=${DSH_VERSION} \
    DSH_TELEMETRY_DISABLED=1

RUN apt-get update \
    && apt-get install -y --no-install-recommends git ca-certificates \
    && rm -rf /var/lib/apt/lists/*

COPY --from=install /usr/local /usr/local

RUN test "$(dsh --version)" = "${DSH_VERSION}"

# `dsh web --host 0.0.0.0` is rejected by the CLI. The home-level patch changes
# only the composed webserver fallback so `docker run -p` can publish the port;
# `--port` still comes from the command line.
RUN mkdir -p /home/node/.dsh /workspace \
    && printf '%s\n' \
        '- id: webserver' \
        '  config:' \
        "    host: !!js ctx.webStartup.host ?? '0.0.0.0'" \
        '    port: !!js ctx.webStartup.port ?? 3080' \
        > /home/node/.dsh/cordis.patch.yml \
    && chown -R node:node /home/node/.dsh /workspace

USER node
WORKDIR /workspace
EXPOSE 3080
ENTRYPOINT ["dsh"]
CMD ["web", "--port", "3080"]
