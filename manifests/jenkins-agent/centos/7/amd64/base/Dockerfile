ARG BASEIMAGE_VERSION=7
FROM amd64/centos:${BASEIMAGE_VERSION} AS base_image

FROM base_image AS build_tools

RUN yum install -y \
        centos-release-scl-rh \
        epel-release && \
    yum update -y && \
    yum install -y \
        devtoolset-6-toolchain \
        devtoolset-6-libasan-devel \
        devtoolset-6-liblsan-devel \
        tar \
        curl \
        openssl \
        ca-certificates \
        zlib-devel && \
    yum clean all && rm -rf /var/cache/yum

ENV M2_HOME /opt/maven
RUN mkdir -p ${M2_HOME} && \
    curl -fsSL http://apache.osuosl.org/maven/maven-3/3.6.3/binaries/apache-maven-3.6.3-bin.tar.gz | \
        tar -xzC ${M2_HOME} --strip-components=1

RUN mkdir -p /opt/sbt && \
    curl -fsSL https://dl.bintray.com/sbt/native-packages/sbt/0.13.13/sbt-0.13.13.tgz | \
    tar -xzC /opt/sbt --strip-components=1

RUN mkdir -p /opt/cmake && \
    curl -fsSL https://cmake.org/files/v3.15/cmake-3.15.5-Linux-x86_64.tar.gz | \
    tar -xzC /opt/cmake --strip-components=1

RUN source /opt/rh/devtoolset-6/enable && \
    curl -fsSL https://github.com/google/protobuf/releases/download/v3.5.1/protobuf-cpp-3.5.1.tar.gz \
    | tar xz && \
    cd protobuf-3.5.1 && \
    ./configure --prefix=/opt/protobuf && \
    make -j2 && \
    make install && \
    cd .. && \
    rm -rf protobuf-3.5.1

FROM base_image AS base_builder_image

COPY --from=build_tools /opt /opt

RUN yum install -y \
        centos-release-scl-rh \
        epel-release && \
    yum update -y && \
    yum install -y \
        devtoolset-6-toolchain \
        devtoolset-6-libasan-devel \
        devtoolset-6-liblsan-devel \
        devtoolset-7-toolchain \
        devtoolset-7-libasan-devel \
        devtoolset-7-liblsan-devel \
        tar \
        wget \
        curl \
        openssl \
        ca-certificates \
        git \
        rpm-build \
        java-1.8.0-openjdk-devel \
        openblas-devel.x86_64 \
        openblas-static.x86_64 \
        # Required for libnd4j CPU tests
        sudo \
        # Required for libnd4j CPU tests (minifier)
        which \
        # Required for Datavec NativeImageLoader
        gtk2-devel \
        # Required for DL4J docs generation
        python-argparse \
        # Required for Datavec-python
        python-pip \
        python36 \
        python36-devel \
        python36-setuptools && \
    yum clean all && rm -rf /var/cache/yum && \
    ln -s /opt/maven/bin/mvn /usr/bin/mvn

# Required for Datavec-python
RUN pip install --upgrade pip && \
    pip install --no-cache-dir Cython --install-option="--no-cython-compile" && \
    easy_install-3.6 pip && \
    pip3 install --upgrade pip && \
    pip3 install --no-cache-dir numpy

ENV HOME /home/jenkins

RUN groupadd jenkins -g 1000 && useradd -d ${HOME} -u 1000 -g 1000 -m jenkins

USER jenkins

WORKDIR ${HOME}

# Not working...
RUN echo 'source /opt/rh/devtoolset-7/enable' >> "${HOME}/.bashrc"

ENV PATH=/opt/sbt/bin:/opt/cmake/bin:/opt/protobuf/bin:${PATH} \
    JAVA_OPTS="-XX:+UnlockExperimentalVMOptions -XX:+UseCGroupMemoryLimitForHeap ${JAVA_OPTS}" \
    PROTOBUF_HOME=/opt/protobuf

CMD ["cat"]
