FROM debian:stable
# Requirements for VSCode
RUN apt update && apt install -y --no-install-recommends curl git libc6 libstdc++6 ca-certificates openssh-server

# Install VS Code
RUN \
curl -Lk 'https://code.visualstudio.com/sha/download?build=stable&os=cli-alpine-x64' --output vscode_cli.tar.gz && \
tar -xf vscode_cli.tar.gz && \
rm -r vscode_cli.tar.gz

# Personal requirements
RUN apt install --no-install-recommends -y wget python3 openjdk-17-jdk-headless

RUN mkdir /vscode && mkdir /data
ENV VSCODE_CLI_DATA_DIR=/vscode/cli

WORKDIR /data
ENTRYPOINT /code tunnel --accept-server-license-terms