FROM markusma/confd:0.16-ubuntu-bionic-s6

# gcc for cgo
RUN apt-get update && apt-get install -y \
    g++ gcc libc6-dev make pkg-config \
    --no-install-recommends \
  && rm -rf /var/lib/apt/lists/*

ENV PATH /usr/local/go/bin:$PATH
ENV GOLANG_VERSION 1.14.4

RUN set -eux; \
  \
# this "case" statement is generated via "update.sh"
  dpkgArch="$(dpkg --print-architecture)"; \
  case "${dpkgArch##*-}" in \
    amd64) goRelArch='linux-amd64'; goRelSha256='aed845e4185a0b2a3c3d5e1d0a35491702c55889192bb9c30e67a3de6849c067' ;; \
    armhf) goRelArch='linux-armv6l'; goRelSha256='e20211425b3f797ca6cd5e9a99ab6d5eaf1b009d08d19fc8a7835544fa58c703' ;; \
    arm64) goRelArch='linux-arm64'; goRelSha256='05dc46ada4e23a1f58e72349f7c366aae2e9c7a7f1e7653095538bc5bba5e077' ;; \
    i386) goRelArch='linux-386'; goRelSha256='4179f406ea0efd455a8071eaaaf1dea92cac5c17aab89fbad18ea2a37623c810' ;; \
    ppc64el) goRelArch='linux-ppc64le'; goRelSha256='b335f85bc935ca3f553ad1bac37da311aaec887ffd8a48cb58a0abb0d8adf324' ;; \
    s390x) goRelArch='linux-s390x'; goRelSha256='17f2ae0bae968b3d909daabc5cc4a37471ddb70ec49076b78702291e6772d71a' ;; \
    *) goRelArch='src'; goRelSha256='7011af3bbc2ac108d1b82ea8abb87b2e63f78844f0259be20cde4d42c5c40584'; \
      echo >&2; echo >&2 "warning: current architecture ($dpkgArch) does not have a corresponding Go binary release; will be building from source"; echo >&2 ;; \
  esac; \
  \
  url="https://golang.org/dl/go${GOLANG_VERSION}.${goRelArch}.tar.gz"; \
  wget -O go.tgz "$url" --progress=dot:giga; \
  echo "${goRelSha256} *go.tgz" | sha256sum -c -; \
  tar -C /usr/local -xzf go.tgz; \
  rm go.tgz; \
  \
# https://github.com/golang/go/issues/38536#issuecomment-616897960
  if [ "$goRelArch" = 'src' ]; then \
    savedAptMark="$(apt-mark showmanual)"; \
    apt-get update; \
    apt-get install -y --no-install-recommends golang-go; \
    \
    goEnv="$(go env | sed -rn -e '/^GO(OS|ARCH|ARM|386)=/s//export \0/p')"; \
    eval "$goEnv"; \
    [ -n "$GOOS" ]; \
    [ -n "$GOARCH" ]; \
    ( \
      cd /usr/local/go/src; \
      ./make.bash; \
    ); \
    \
    apt-mark auto '.*' > /dev/null; \
    apt-mark manual $savedAptMark > /dev/null; \
    apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false; \
    rm -rf /var/lib/apt/lists/*; \
    \
# pre-compile the standard library, just like the official binary release tarballs do
    go install std; \
# go install: -race is only supported on linux/amd64, linux/ppc64le, linux/arm64, freebsd/amd64, netbsd/amd64, darwin/amd64 and windows/amd64
#    go install -race std; \
    \
# remove a few intermediate / bootstrapping files the official binary release tarballs do not contain
    rm -rf \
      /usr/local/go/pkg/*/cmd \
      /usr/local/go/pkg/bootstrap \
      /usr/local/go/pkg/obj \
      /usr/local/go/pkg/tool/*/api \
      /usr/local/go/pkg/tool/*/go_bootstrap \
      /usr/local/go/src/cmd/dist/dist \
    ; \
  fi; \
  \
  go version

ENV GOPATH /go
ENV PATH $GOPATH/bin:$PATH
RUN mkdir -p "$GOPATH/src" "$GOPATH/bin" && chmod -R 777 "$GOPATH"
WORKDIR $GOPATH
