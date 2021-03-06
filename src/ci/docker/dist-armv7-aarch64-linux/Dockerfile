FROM ubuntu:16.04

RUN apt-get update && apt-get install -y --no-install-recommends \
  automake \
  bison \
  bzip2 \
  ca-certificates \
  cmake \
  curl \
  file \
  flex \
  g++ \
  gawk \
  gdb \
  git \
  gperf \
  help2man \
  libncurses-dev \
  libtool-bin \
  make \
  patch \
  python2.7 \
  sudo \
  texinfo \
  wget \
  xz-utils \
  libssl-dev \
  pkg-config

RUN curl -o /usr/local/bin/sccache \
      https://s3.amazonaws.com/rust-lang-ci/rust-ci-mirror/2017-02-24-sccache-x86_64-unknown-linux-gnu && \
      chmod +x /usr/local/bin/sccache

RUN curl -OL https://github.com/Yelp/dumb-init/releases/download/v1.2.0/dumb-init_1.2.0_amd64.deb && \
    dpkg -i dumb-init_*.deb && \
    rm dumb-init_*.deb
ENTRYPOINT ["/usr/bin/dumb-init", "--"]

# Ubuntu 16.04 (this contianer) ships with make 4, but something in the
# toolchains we build below chokes on that, so go back to make 3
RUN curl https://ftp.gnu.org/gnu/make/make-3.81.tar.gz | tar xzf - && \
      cd make-3.81 && \
      ./configure --prefix=/usr && \
      make && \
      make install && \
      cd .. && \
      rm -rf make-3.81

RUN curl http://crosstool-ng.org/download/crosstool-ng/crosstool-ng-1.22.0.tar.bz2 | \
      tar xjf - && \
      cd crosstool-ng && \
      ./configure --prefix=/usr/local && \
      make -j$(nproc) && \
      make install && \
      cd .. && \
      rm -rf crosstool-ng

RUN groupadd -r rustbuild && useradd -m -r -g rustbuild rustbuild
RUN mkdir /x-tools && chown rustbuild:rustbuild /x-tools
USER rustbuild
WORKDIR /tmp

COPY armv7-linux-gnueabihf.config /tmp/
COPY armv7-linux-gnueabihf.config aarch64-linux-gnu.config build-toolchains.sh /tmp/
RUN ./build-toolchains.sh

USER root

ENV PATH=$PATH:/x-tools/aarch64-unknown-linux-gnueabi/bin
ENV PATH=$PATH:/x-tools/armv7-unknown-linux-gnueabihf/bin

ENV CC_aarch64_unknown_linux_gnu=aarch64-unknown-linux-gnueabi-gcc \
    AR_aarch64_unknown_linux_gnu=aarch64-unknown-linux-gnueabi-ar \
    CXX_aarch64_unknown_linux_gnu=aarch64-unknown-linux-gnueabi-g++ \
    CC_armv7_unknown_linux_gnueabihf=armv7-unknown-linux-gnueabihf-gcc \
    AR_armv7_unknown_linux_gnueabihf=armv7-unknown-linux-gnueabihf-ar \
    CXX_armv7_unknown_linux_gnueabihf=armv7-unknown-linux-gnueabihf-g++

ENV HOSTS=armv7-unknown-linux-gnueabihf
ENV HOSTS=$HOSTS,aarch64-unknown-linux-gnu

ENV RUST_CONFIGURE_ARGS --host=$HOSTS --enable-extended
ENV SCRIPT python2.7 ../x.py dist --host $HOSTS --target $HOSTS
