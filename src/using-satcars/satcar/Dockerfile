
# This is basically only daprd with an added nginx reverse-proxy 
# Expected size:
# Unoptimized : 110MB
# Optimized   :  32MB

# Cache the download step to prevent it from slowing down the release 
FROM alpine:latest as dapr-downloader
WORKDIR /app
# Dapr runtime version to use 
ARG DAPR_VERSION=1.5.0
# Wether to optimize the daprd binary
ARG OPTIMIZE=0
RUN apk add binutils upx 
ADD https://github.com/dapr/dapr/releases/download/v${DAPR_VERSION}/daprd_linux_amd64.tar.gz .
RUN tar xvf daprd_linux_amd64.tar.gz 
# Size optimizations for daprd. This will most likely erase the stack strace if any problem happen
# Don't do it in debug mode
RUN if [[ "$OPTIMIZE" -eq 1 ]]; then strip /app/daprd && upx --best --lzma /app/daprd; else echo "Bypassing optimizations"; fi;


FROM alpine:latest
WORKDIR /app
COPY nginx.conf /etc/nginx/nginx.conf.template
# COPY config.yaml /config/config.yaml
COPY --chmod=500 --chown=root:root entrypoint.sh /app/
# COPY components /components
# gettext is the apk package containing envsubst used in the entrypoint
RUN apk add --no-cache gettext nginx su-exec &&\
    adduser -HD -s /bin/ash appuser
COPY --from=dapr-downloader --chmod=0500 --chown=appuser:appuser /app/daprd .
ENTRYPOINT [ "sh", "/app/entrypoint.sh" ]

