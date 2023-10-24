FROM debian:stable
RUN apt update && apt install -y curl gpg
RUN curl https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > microsoft.gpg
RUN install -o root -g root -m 644 microsoft.gpg /usr/share/keyrings/microsoft-archive-keyring.gpg
RUN sh -c 'echo "deb [arch=amd64,arm64,armhf signed-by=/usr/share/keyrings/microsoft-archive-keyring.gpg] https://packages.microsoft.com/repos/vscode stable main" > /etc/apt/sources.list.d/vscode.list'
RUN apt update && apt install -y code python3 wget git libc6 libstdc++6 ca-certificates tar openssh-server
RUN python3 -m ensurepip
RUN pip3 install --no-cache --upgrade pip setuptools
ENV DONT_PROMPT_WSL_INSTALL=true
RUN useradd -ms /bin/bash vscode
USER vscode
WORKDIR /home/vscode
ENTRYPOINT code tunnel --accept-server-license-terms