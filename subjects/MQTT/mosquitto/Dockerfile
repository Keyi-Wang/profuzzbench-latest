FROM ubuntu:20.04

# Install common dependencies
ENV DEBIAN_FRONTEND=noninteractive
RUN sed -i 's/archive.ubuntu.com/mirrors.aliyun.com/g' /etc/apt/sources.list
RUN sed -i 's/security.ubuntu.com/mirrors.aliyun.com/g' /etc/apt/sources.list
RUN apt-get -y update && \
    apt-get -y install sudo \ 
    apt-utils \
    build-essential \
    openssl \
    clang \
    graphviz-dev \
    git \
    autoconf \
    libgnutls28-dev \
    libssl-dev \
    libcap-dev\
    libwebsockets-dev \
    uuid-dev \
    docbook-xsl \
    docbook \
    xsltproc \
    llvm \
    python3-pip \
    nano \
    net-tools \
    vim \
    gdb \
    netcat \
    strace \
    wget

# Add a new user ubuntu, pass: ubuntu
RUN groupadd ubuntu && \
    useradd -rm -d /home/ubuntu -s /bin/bash -g ubuntu -G sudo -u 1000 ubuntu -p "$(openssl passwd -1 ubuntu)"

RUN chmod 777 /tmp

RUN pip3 install gcovr==4.2

# Use ubuntu as default username
USER ubuntu
WORKDIR /home/ubuntu

# Import environment variable to pass as parameter to make (e.g., to make parallel builds with -j)
ARG MAKE_OPT

# Set up fuzzers
RUN git clone https://github.com/Keyi-Wang/llm-aflnet.git aflnet && \
    cd aflnet && \
    make clean all $MAKE_OPT && \
    cd llvm_mode && make $MAKE_OPT

# Set up environment variables for AFLNet
ENV WORKDIR="/home/ubuntu/experiments"
ENV AFLNET="/home/ubuntu/aflnet"
ENV PATH="${PATH}:${AFLNET}:/home/ubuntu/.local/bin:${WORKDIR}"
ENV AFL_PATH="${AFLNET}"
ENV AFL_I_DONT_CARE_ABOUT_MISSING_CRASHES=1 \
    AFL_SKIP_CPUFREQ=1 \
    AFL_NO_AFFINITY=1


# Set up environment variables for ASAN
env ASAN_OPTIONS='abort_on_error=1:symbolize=0:detect_leaks=0:detect_stack_use_after_return=1:detect_container_overflow=0:poison_array_cookie=0:malloc_fill_byte=0:max_malloc_fill_size=16777216'


RUN mkdir $WORKDIR

COPY --chown=ubuntu:ubuntu mosquitto.patch ${WORKDIR}/mosquitto.patch

# Set up ProFTPD for fuzzing
RUN cd ${WORKDIR} && \
    git clone https://github.com/eclipse/mosquitto.git && \ 
    cd mosquitto && \
    git checkout c6c6850 && \
    patch -p1 < ${WORKDIR}/mosquitto.patch && \
    CFLAGS="-g -O0 -fsanitize=address -fno-omit-frame-pointer" LDFLAGS="-g -O0 -fsanitize=address -fno-omit-frame-pointer" CC=afl-gcc AFL_USE_ASAN=1 make clean all WITH_TLS=no WITH_TLS_PSK:=no WITH_STATIC_LIBRARIES=yes WITH_DOCS=no WITH_CJSON=no WITH_EPOLL:=no $MAKE_OPT

# Set up ProFTPD for coverage analysis
RUN cd ${WORKDIR} && \
    git clone https://github.com/eclipse/mosquitto.git mosquitto-gcov && \ 
    cd mosquitto-gcov && \
    git checkout c6c6850 && \
    patch -p1 < ${WORKDIR}/mosquitto.patch && \
    make clean all \
    CC=gcc \
    CFLAGS='-fprofile-arcs -ftest-coverage' \
    LDFLAGS='-fprofile-arcs -ftest-coverage' \
    WITH_TLS=no WITH_TLS_PSK=no WITH_STATIC_LIBRARIES=yes WITH_DOCS=no WITH_CJSON=no WITH_EPOLL=no $MAKE_OPT


COPY --chown=ubuntu:ubuntu in-mqtt ${WORKDIR}/in-mqtt
COPY --chown=ubuntu:ubuntu run.sh ${WORKDIR}/run
COPY --chown=ubuntu:ubuntu cov_script.sh ${WORKDIR}/cov_script

