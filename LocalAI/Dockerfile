ARG BASE_IMAGE=ubuntu:22.04
ARG GRPC_BACKENDS=backend-assets/grpc/llama-cpp
ARG GRPC_VERSION=v1.58.0
ARG HEALTHCHECK_ENDPOINT=http://localhost:8080/readyz

############################################
############################################

FROM ${BASE_IMAGE} AS builder
RUN apt-get update && apt-get install -y \
    git \
    build-essential \
    cmake \
    curl \
    unzip

############################################
############################################

FROM builder AS localai-source
WORKDIR /
RUN git clone https://github.com/mudler/LocalAI /build

############################################
############################################

FROM builder AS grpc

ARG MAKEFLAGS
ARG GRPC_VERSION

WORKDIR /build
RUN git clone --recurse-submodules --jobs 4 -b ${GRPC_VERSION} --depth 1 --shallow-submodules https://github.com/grpc/grpc

WORKDIR /build/grpc
RUN mkdir -p cmake/build
WORKDIR /build/grpc/cmake/build
RUN cmake -DCMAKE_INSTALL_PREFIX=/build/grpc/output -DgRPC_INSTALL=ON -DgRPC_BUILD_TESTS=OFF ../..
RUN make
RUN make install

############################################
############################################

FROM builder AS requirements-core

USER root

ARG GO_VERSION=1.21.7
ARG BUILD_TYPE
ARG CUDA_MAJOR_VERSION=11
ARG CUDA_MINOR_VERSION=7
ARG TARGETARCH
ARG TARGETVARIANT

ENV BUILD_TYPE=${BUILD_TYPE}
ENV DEBIAN_FRONTEND=noninteractive
#ENV EXTERNAL_GRPC_BACKENDS="coqui:/build/backend/python/coqui/run.sh,huggingface-embeddings:/build/backend/python/sentencetransformers/run.sh,petals:/build/backend/python/petals/run.sh,transformers:/build/backend/python/transformers/run.sh,sentencetransformers:/build/backend/python/sentencetransformers/run.sh,autogptq:/build/backend/python/autogptq/run.sh,bark:/build/backend/python/bark/run.sh,diffusers:/build/backend/python/diffusers/run.sh,exllama:/build/backend/python/exllama/run.sh,vall-e-x:/build/backend/python/vall-e-x/run.sh,vllm:/build/backend/python/vllm/run.sh,mamba:/build/backend/python/mamba/run.sh,exllama2:/build/backend/python/exllama2/run.sh,transformers-musicgen:/build/backend/python/transformers-musicgen/run.sh,parler-tts:/build/backend/python/parler-tts/run.sh"
ENV EXTERNAL_GRPC_BACKENDS=

ARG GO_TAGS=""

RUN apt-get install -y \
        ca-certificates \
        python3-pip \
        unzip \
        #libopenblas-dev \
        libopencv-dev \
    && \
    apt-get clean

# Install Go
RUN curl -L -s https://go.dev/dl/go$GO_VERSION.linux-$TARGETARCH.tar.gz | tar -C /usr/local -xz
ENV PATH $PATH:/usr/local/go/bin

# Install grpc compilers
ENV PATH $PATH:/root/go/bin
RUN go install google.golang.org/protobuf/cmd/protoc-gen-go@latest && \
    go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@latest

# Install protobuf (the version in 22.04 is too old)
RUN curl -L -s https://github.com/protocolbuffers/protobuf/releases/download/v26.1/protoc-26.1-linux-x86_64.zip -o protoc.zip && \
    unzip -j -d /usr/local/bin protoc.zip bin/protoc && \
    rm protoc.zip

# Install grpcio-tools (the version in 22.04 is too old)
RUN pip install --user grpcio-tools

# Set up OpenCV
RUN ln -s /usr/include/opencv4/opencv2 /usr/include/opencv2

COPY --from=localai-source --chmod=644 /build/custom-ca-certs/* /usr/local/share/ca-certificates/
RUN update-ca-certificates

RUN echo "Target Architecture: $TARGETARCH"
RUN echo "Target Variant: $TARGETVARIANT"

############################################
############################################

FROM requirements-core AS localai-builder
ARG GO_TAGS=""
ARG GRPC_BACKENDS
ARG MAKEFLAGS
ARG HEALTHCHECK_ENDPOINT

COPY --from=localai-source /build /build
COPY root/ .

ENV GRPC_BACKENDS=${GRPC_BACKENDS}
ENV GO_TAGS=${GO_TAGS}
ENV MAKEFLAGS=${MAKEFLAGS}
ENV NVIDIA_DRIVER_CAPABILITIES=compute,utility
ENV NVIDIA_REQUIRE_CUDA="cuda>=${CUDA_MAJOR_VERSION}.0"
ENV NVIDIA_VISIBLE_DEVICES=all
ENV HEALTHCHECK_ENDPOINT=${HEALTHCHECK_ENDPOINT}

ENV REBUILD=false

WORKDIR /build
RUN echo "GRPC_BACKENDS ${GRPC_BACKENDS}"
RUN make prepare

COPY --from=grpc /build/grpc/output /usr/local
ENV PATH=/usr/local/bin:$PATH
RUN make build

# Cleanup build tools
# I need to move some stuff to other images to
# reduce the final image size, but this is for
# security. Should definitely do more though.
RUN rm -rf \
    /build/grpc \
    /root/go

RUN apt-get remove -y \
    git \
    build-essential \
    cmake \
    curl \
    unzip

RUN mkdir -p /build/models

# Define the health check command
HEALTHCHECK --interval=1m --timeout=10m --retries=10 \
  CMD curl -f $HEALTHCHECK_ENDPOINT || exit 1
  
VOLUME /build/models
EXPOSE 8080
ENTRYPOINT [ "/build/entrypoint.sh" ]