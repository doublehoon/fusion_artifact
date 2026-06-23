#
# Expected artifact layout on the host:
#
#   artifact-root/
#     .dockerignore
#     docker/
#       Dockerfile
#     fusion/
#       ... cloned repository contents ...
#
# Reviewer workflow:
#
#   cd artifact-root
#   git clone https://github.com/doublehoon/fusion.git fusion
#   docker build -t fusion-issta -f docker/Dockerfile .
#   docker run --rm -it fusion-issta
#
# This Dockerfile expects the Docker build context to be `artifact-root/`.
# It therefore copies `fusion/` from that build context into the image.
#
FROM ubuntu:22.04

ENV DEBIAN_FRONTEND=noninteractive
ENV SBT_VERSION=1.10.0
ENV DOTNET_SDK_VERSION=8.0.125
ENV BOOGIE_VERSION=3.5.5

ENV LANG=C.UTF-8 \
    LC_ALL=C.UTF-8 \
    PYTHONUNBUFFERED=1 \
    PIP_NO_CACHE_DIR=1 \
    JAVA_HOME=/opt/java/openjdk \
    Z3_EXE=/usr/bin/z3 \
    DOTNET_ROOT=/usr/local/dotnet \
    BOOGIE_EXE=/usr/local/bin/boogie \
    DOTNET_SYSTEM_GLOBALIZATION_INVARIANT=true

# Core artifact dependencies:
# - Ubuntu 22.04 base
# - OpenJDK 17 + sbt for building Gobra
# - Python 3 installed from Ubuntu 22.04 apt repositories
# - Z3 installed from Ubuntu 22.04 apt repositories
# - Boogie 3.5.5 to match the current local environment
# - procps for `pkill`, which the experiment driver may use for z3 cleanup
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    bash \
    ca-certificates \
    curl \
    git \
    libicu70 \
    openjdk-17-jdk-headless \
    procps \
    python3 \
    python3-pip \
    python3-venv \
    unzip \
    zip \
    z3 && \
    mkdir -p /opt/java && \
    ln -s "$(dirname "$(dirname "$(readlink -f "$(which javac)")")")" "${JAVA_HOME}" && \
    test -x "${JAVA_HOME}/bin/java" && \
    ln -sf /usr/bin/python3 /usr/local/bin/python && \
    ln -sf /usr/bin/python3 /usr/local/bin/python3 && \
    ln -sf /usr/bin/pip3 /usr/local/bin/pip && \
    ln -sf /usr/bin/pip3 /usr/local/bin/pip3 && \
    java --version && \
    javac --version && \
    python --version && \
    pip --version && \
    z3 -version && \
    rm -rf /var/lib/apt/lists/*

RUN curl -fsSL -o /tmp/sbt.tgz \
    https://github.com/sbt/sbt/releases/download/v${SBT_VERSION}/sbt-${SBT_VERSION}.tgz && \
    tar -xzf /tmp/sbt.tgz -C /opt && \
    ln -s /opt/sbt/bin/sbt /usr/local/bin/sbt && \
    rm /tmp/sbt.tgz

RUN curl -fsSL -o /tmp/dotnet-install.sh https://dot.net/v1/dotnet-install.sh && \
    chmod +x /tmp/dotnet-install.sh && \
    /tmp/dotnet-install.sh --install-dir "${DOTNET_ROOT}" --version "${DOTNET_SDK_VERSION}" && \
    ln -s "${DOTNET_ROOT}/dotnet" /usr/local/bin/dotnet && \
    dotnet --version && \
    dotnet tool install --tool-path /usr/local/bin Boogie --version "${BOOGIE_VERSION}" && \
    boogie /version && \
    rm -f /tmp/dotnet-install.sh

WORKDIR /workspace/fusion
COPY fusion /workspace/fusion

RUN mkdir -p /workspace/fusion/output/experiments

RUN cd /workspace/fusion/gobra && \
    sbt compile && \
    sbt assembly

RUN java --version && \
    javac --version && \
    test -x "${JAVA_HOME}/bin/java" && \
    sbt --script-version && \
    python --version && \
    pip --version && \
    z3 -version && \
    boogie /version

WORKDIR /workspace/fusion
CMD ["/bin/bash"]