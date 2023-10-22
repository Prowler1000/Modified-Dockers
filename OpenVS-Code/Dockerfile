FROM gitpod/openvscode-server:latest

ENV OPENVSCODE_SERVER_ROOT="/home/.openvscode-server"
ENV OPENVSCODE="${OPENVSCODE_SERVER_ROOT}/bin/openvscode-server"

USER root
RUN
USER openvscode-server

SHELL ["/bin/bash", "-c"]
RUN \
	urls=(\
		https://marketplace.visualstudio.com/_apis/public/gallery/publishers/GitHub/vsextensions/copilot/1.128.504/vspackage \
	)\
	&& tdir=/tmp/exts && mkdir -p "${tdir}" && cd "${tdir}" \
	&& wget "${urls[@]" && \
	exts=(\
		#test \
		"${tdir}"/* \
	)\
	&& for ext in "${exts[@]}"; do ${OPENVSCODE} --install-extension "${ext}"; done