FROM ubuntu:20.04

# update ca-certificates
RUN set -x \
    && apt-get update \
    && apt-get install -y --no-install-recommends \
        ca-certificates \
    && apt-get clean && rm -rf /var/cache/apt/archives/* /var/lib/apt/lists/*

# timezone
RUN set -x \
    && apt-get update \
    && apt-get install -y --no-install-recommends \
        tzdata \
    && apt-get clean && rm -rf /var/cache/apt/archives/* /var/lib/apt/lists/* \
    && ln -sf /usr/share/zoneinfo/Asia/Tokyo /etc/localtime \
    && echo 'Asia/Tokyo' >/etc/timezone

# vscode user
RUN set -x \
    && apt-get update \
    && apt-get install -y --no-install-recommends \
        sudo \
    && apt-get clean && rm -rf /var/cache/apt/archives/* /var/lib/apt/lists/* \
    && echo "vscode ALL=(ALL) NOPASSWD:ALL" >>/etc/sudoers.d/ALL \
    && groupadd \
        --gid 5000 \
        vscode \
    && useradd \
        --uid 5000 \
        --gid 5000 \
        --home-dir /home/vscode \
        --create-home \
        --shell /bin/bash \
        vscode

# common dev tools
RUN set -x \
    && apt-get update \
    && apt-get install -y --no-install-recommends \
        bash-completion \
        curl \
        git \
        gnupg2 \
        iputils-ping \
        jq \
        less \
        net-tools \
        openssh-client \
        tar \
        time \
        unzip \
        xz-utils \
        zip \
    && apt-get clean && rm -rf /var/cache/apt/archives/* /var/lib/apt/lists/*

# golang
ARG GO_VERSION=1.20.4
RUN set -x \
    && apt-get update \
    && apt-get install -y --no-install-recommends \
        build-essential \
        curl \
        git \
    && apt-get clean && rm -rf /var/cache/apt/archives/* /var/lib/apt/lists/* \
    && curl -fsSL "https://go.dev/dl/go${GO_VERSION}.linux-amd64.tar.gz" | tar -xz -C /usr/local
ENV GOROOT=/usr/local/go \
    GOPATH=/home/vscode/go \
    PATH=/home/vscode/go/bin:/usr/local/go/bin:${PATH}

# nodejs
ARG NODE_VERSION=20.1.0
RUN set -x \
    && apt-get update \
    && apt-get install -y --no-install-recommends \
        curl \
    && apt-get clean && rm -rf /var/cache/apt/archives/* /var/lib/apt/lists/* \
    && mkdir -p /usr/local/lib/nodejs \
    && curl -fsSL "https://nodejs.org/dist/v${NODE_VERSION}/node-v${NODE_VERSION}-linux-x64.tar.xz" | tar xJ -C /usr/local/lib/nodejs
ENV PATH=/usr/local/lib/nodejs/node-v${NODE_VERSION}-linux-x64/bin:${PATH}

USER vscode

# hugo
ARG HUGO_VERSION=0.111.3
RUN set -x \
    && CGO_ENABLED=1 go install -tags extended github.com/gohugoio/hugo@v${HUGO_VERSION}

