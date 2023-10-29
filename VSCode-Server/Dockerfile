FROM debian:stable
# Requirements for VSCode
RUN apt update && apt install -y --no-install-recommends curl git libc6 libstdc++6 ca-certificates openssh-server

# Install VS Code
RUN \
curl -Lk 'https://code.visualstudio.com/sha/download?build=insider&os=cli-alpine-x64' --output vscode_cli.tar.gz && \
tar -xf vscode_cli.tar.gz -C /usr/bin && \
rm -r vscode_cli.tar.gz

# Personal requirements
RUN apt install --no-install-recommends -y wget python3 openjdk-17-jdk-headless

RUN mkdir /vscode-cli && mkdir /data
ENV VSCODE_CLI_DATA_DIR=/vscode-cli
ENV VSCODE_CLI_USE_FILE_KEYCHAIN=1
ENV TUNNEL_NAME=docker_tunnel
ENV LOG_LEVEL=debug

WORKDIR /data
ENTRYPOINT code-insiders tunnel --name $TUNNEL_NAME --log $LOG_LEVEL --accept-server-license-terms