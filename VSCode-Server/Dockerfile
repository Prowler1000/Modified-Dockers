FROM debian:stable
# Requirements for VSCode
RUN apt update && apt install -y curl git libc6 libstdc++6 ca-certificates openssh-server

# Install VSCode CLI
RUN \
curl -Lk 'https://code.visualstudio.com/sha/download?build=stable&os=cli-alpine-x64' --output vscode_cli.tar.gz && \
tar -xf vscode_cli.tar.gz

# Personal requirements
RUN apt install -y wget

# Prep VSCode and define entry
ENV DONT_PROMPT_WSL_INSTALL=true

RUN useradd -m vscode && mv /code /home/vscode/code
USER vscode
WORKDIR /home/vscode
ENTRYPOINT ./code tunnel --accept-server-license-terms