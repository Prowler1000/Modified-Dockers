FROM debian:stable
# Requirements for VSCode
RUN apt update && apt install -y --no-install-recommends curl git gpg libc6 libstdc++6 ca-certificates openssh-server

# Install VSCode CLI
#RUN \
#curl -Lk 'https://code.visualstudio.com/sha/download?build=stable&os=cli-alpine-x64' --output vscode_cli.tar.gz && \
#tar -xf vscode_cli.tar.gz && \
#rm -r vscode_cli.tar.gz
RUN \
curl -Lk 'https://code.visualstudio.com/sha/download?build=stable&os=linux-deb-x64' --output vscode.deb && \
apt install -y --no-install-recommends /vscode.deb && \
rm /vscode.deb


# Personal requirements
RUN apt install --no-install-recommends -y wget

# Prep VSCode and define entry
ENV DONT_PROMPT_WSL_INSTALL=true
ENV VSCODE_CLI_DATA_DIR=/vscode

RUN useradd -m vscode
RUN mkdir /vscode && chown -R vscode: /vscode
USER vscode
WORKDIR /vscode
ENTRYPOINT code tunnel --accept-server-license-terms