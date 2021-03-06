FROM openjdk:8-jdk

RUN java -version
RUN echo "JAVA_HOME=${JAVA_HOME}"
ENV JAVA8_HOME="${JAVA_HOME}"
RUN echo "JAVA8_HOME=${JAVA8_HOME}"

### Grakn, Graql installation

RUN echo "Building the Open Source version (AGPL license), go to https://github.com/graknlabs for further details"
RUN apt-get update && apt-get install -y wget unzip pv \
    --no-install-recommends \
    && rm -r /var/lib/apt/lists/*

ARG GRAKN_VERSION
ENV GRAKN_VERSION=${GRAKN_VERSION:-1.5.7}
ENV WORKDIR=/home/grakn
WORKDIR ${WORKDIR}

ENV GRAKN_CORE_LINUX=grakn-core-all-linux
RUN wget https://github.com/graknlabs/grakn/releases/download/${GRAKN_VERSION}/${GRAKN_CORE_LINUX}-${GRAKN_VERSION}.tar.gz --progress=bar:force 2>&1 | tail -f -n +3
RUN tar zxvf ${GRAKN_CORE_LINUX}-${GRAKN_VERSION}.tar.gz | pv -l >/dev/null
RUN rm ${GRAKN_CORE_LINUX}-${GRAKN_VERSION}.tar.gz
RUN ./${GRAKN_CORE_LINUX}-${GRAKN_VERSION}/grakn version

### GraalVM installation
ARG GRAALVM_VERSION
ENV GRAALVM_VERSION="${GRAALVM_VERSION:-19.1.1}"
RUN wget https://github.com/oracle/graal/releases/download/vm-${GRAALVM_VERSION}/graalvm-ce-linux-amd64-${GRAALVM_VERSION}.tar.gz --progress=bar:force 2>&1 | tail -f -n +3
RUN tar xvzf graalvm-ce-linux-amd64-${GRAALVM_VERSION}.tar.gz -C /usr/local | pv -l >/dev/null

ENV GRAALVM_HOME="/usr/local/graalvm-ce-${GRAALVM_VERSION}"
RUN echo "GRAALVM_HOME=${GRAALVM_HOME}"
RUN ${GRAALVM_HOME}/bin/java -version
RUN rm graalvm-ce-linux-amd64-${GRAALVM_VERSION}.tar.gz
RUN rm -fr graalvm-ce-${GRAALVM_VERSION}

### Downloading native-image
RUN ${GRAALVM_HOME}/bin/gu install native-image

### Install bazel
RUN apt-get update && apt-get install -qy pkg-config zip g++ zlib1g-dev unzip python3 \
    --no-install-recommends \
    && rm -r /var/lib/apt/lists/*
ENV BAZEL_VERSION=0.26.1
RUN curl -J -L -O  https://github.com/bazelbuild/bazel/releases/download/${BAZEL_VERSION}/bazel-${BAZEL_VERSION}-installer-linux-x86_64.sh
RUN chmod +x bazel-${BAZEL_VERSION}-installer-linux-x86_64.sh
RUN ./bazel-${BAZEL_VERSION}-installer-linux-x86_64.sh
RUN export PATH=$HOME/bin:${PATH}

RUN apt-get update && apt-get install -qy vim \
    --no-install-recommends \
    && rm -r /var/lib/apt/lists/*
RUN mkdir -p .cache

COPY startGraknAndGraql.sh ./startGraknAndGraql.sh
COPY runPerformanceBenchmark.sh ./runPerformanceBenchmark.sh
COPY mergeJson.sh ./mergeJson.sh
COPY buildUberJar.sh ./buildUberJar.sh
COPY 0001-Build-a-distribution-with-uber-jars.patch ./0001-Build-a-distribution-with-uber-jars.patch

RUN groupadd -r user_grp && useradd -r -g user_grp grakn
RUN chown -R grakn:user_grp .

USER grakn

COPY .bashrc ${WORKDIR}/.bashrc

ENTRYPOINT ["bash", "-c", "./startGraknAndGraql.sh"]