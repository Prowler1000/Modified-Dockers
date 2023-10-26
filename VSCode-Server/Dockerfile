FROM debian:stable

# Requirements for VSCode
RUN apt update && apt install -y --no-install-recommends curl git libc6 libstdc++6 ca-certificates openssh-server

# Install VS Code
RUN \
curl -Lk 'https://code.visualstudio.com/sha/download?build=stable&os=cli-alpine-x64' --output vscode_cli.tar.gz && \
tar -xf vscode_cli.tar.gz && \
rm -r vscode_cli.tar.gz

# Personal requirements
RUN apt install --no-install-recommends -y wget

ENV VSCODE_CLI_DATA_DIR=/vscode/cli
ENV USER=vscode
ENV UID=1199

RUN \
mkdir /userdata && mkdir /vscode && \
adduser --uid $UID $USER && \
chown -R :$USER /userdata && \
chown -R :$USER /vscode

USER $USER
WORKDIR /userdata
ENTRYPOINT /code tunnel --accept-server-license-terms