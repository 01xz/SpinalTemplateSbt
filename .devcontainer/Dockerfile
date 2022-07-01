ARG IMAGE="ubuntu:20.04"

FROM ${IMAGE} as common

RUN sed -i 's/cn.archive.ubuntu.com/mirrors.ustc.edu.cn/g' /etc/apt/sources.list \
    && apt update && apt upgrade -y && apt install -y --no-install-recommends \
    apt-utils apt-transport-https ca-certificates git make \
    && apt autoclean && apt clean && apt -y autoremove \
    && update-ca-certificates

FROM common as build

ARG GIT_PROXY

RUN apt install -y --no-install-recommends \
    perl python3 autoconf clang flex bison ccache gperf \
    libgoogle-perftools-dev numactl perl-doc \
    libfl2 libfl-dev zlibc zlib1g zlib1g-dev \
    && git config --global http.proxy ${GIT_PROXY} \
    && git config --global https.proxy ${GIT_PROXY}

RUN git clone https://github.com/verilator/verilator \
    && cd verilator \
    && git checkout stable \
    && unset VERILATOR_ROOT \
    && autoconf \
    && ./configure --prefix="/usr/local/" \
    && make -j$(nproc) \
    && make install DESTDIR="$(pwd)/install-verilator" \
    && mv install-verilator/usr/local /tmp/verilator \
    && cd ..

RUN git clone https://github.com/steveicarus/iverilog --depth=1 --branch v11-branch \
    && cd iverilog \
    && autoconf \
    && ./configure \
    && make -j$(nproc) \
    && make install DESTDIR="$(pwd)/install-iverilog" \
    && mv install-iverilog/usr/local /tmp/iverilog \
    && cd ..

FROM common as base

ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=$USER_UID

ENV DEBIAN_FRONTEND=noninteractive

RUN apt install -y --no-install-recommends sudo bash \
    perl python3 python-is-python3 clang libfl2 libfl-dev zlibc zlib1g zlib1g-dev \
    ccache libgoogle-perftools-dev numactl gnupg2 openjdk-11-jdk curl locales \
    && sed -i -e 's/# en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/' /etc/locale.gen && locale-gen \
    && echo "deb https://repo.scala-sbt.org/scalasbt/debian all main" | tee /etc/apt/sources.list.d/sbt.list \
    && echo "deb https://repo.scala-sbt.org/scalasbt/debian /" | tee /etc/apt/sources.list.d/sbt_old.list \
    && curl -sL "https://keyserver.ubuntu.com/pks/lookup?op=get&search=0x2EE0EA64E40A89B84B2DF73499E82A75642AC823" | apt-key add \
    && apt update && apt install -y --no-install-recommends sbt scala \
    && apt autoclean && apt clean && apt -y autoremove \
    && rm -rf /var/lib/apt/lists/* \
    && groupadd --gid $USER_GID $USERNAME \
    && useradd --uid $USER_UID --gid $USER_GID -m $USERNAME -s /usr/bin/bash \
    && echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME \
    && chmod 0440 /etc/sudoers.d/$USERNAME

COPY --from=build /tmp/verilator/ /usr/local/
COPY --from=build /tmp/iverilog/ /usr/local/

USER $USERNAME

ENTRYPOINT [ "/usr/bin/bash" ]
