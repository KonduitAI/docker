ARG BASEIMAGE_VERSION=32.2.1-nano-jetpack-4.2.2-base
FROM sshepel/jenkins-agent:${BASEIMAGE_VERSION} AS base_image

FROM base_image AS build_tools

# Set DEBIAN_FRONTEND to skip any interactive post-install configuration steps
ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get -qqy update && \
    apt-get -y --no-install-recommends install \
        curl \
        ca-certificates \
        build-essential && \
    apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

RUN curl -fsSL https://cmake.org/files/v3.15/cmake-3.15.5.tar.gz | \
        tar xz && \
        cd cmake-3.15.5 && \
        ./configure --prefix=/opt/cmake && \
        make -j2 && \
        make install && \
        cd .. && \
        rm -r cmake-3.15.5

FROM base_image AS base_builder_image

COPY --from=build_tools /opt /opt

RUN apt-get -qqy update && \
    apt-get -y --no-install-recommends install \
        wget \
        curl \
        ca-certificates \
        ca-certificates-java \
        software-properties-common \
        git \
        build-essential \
        gnupg-agent \
        dirmngr \
        openjdk-8-jdk-headless \
        libopenblas-dev \
        maven && \
    apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* && \
    update-java-alternatives -s java-1.8.0-openjdk-arm64 && \
    ln -s /opt/cmake/bin/* /usr/local/bin

RUN add-apt-repository ppa:ubuntu-toolchain-r/test && \
    apt-get -qqy update && \
    apt-get -y --no-install-recommends install \
        gcc-8 \
        g++-8 && \
    apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

ENV HOME /home/jenkins

RUN groupadd jenkins -g 1000 && useradd -d ${HOME} -u 1000 -g 1000 -m jenkins

USER jenkins

WORKDIR ${HOME}

RUN mkdir -p .gnupg

ENV PATH=/opt/cmake/bin:${PATH} \
    JAVA_OPTS="-XX:+UnlockExperimentalVMOptions -XX:+UseCGroupMemoryLimitForHeap ${JAVA_OPTS}"

CMD ["cat"]
