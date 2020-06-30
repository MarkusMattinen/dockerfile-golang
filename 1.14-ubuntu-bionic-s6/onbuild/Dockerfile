FROM markusma/golang:1.14-ubuntu-bionic-s6

RUN mkdir -p /go/src/app
WORKDIR /go/src/app

ONBUILD COPY . /go/src/app
ONBUILD RUN go get -v -d
ONBUILD RUN go install -v
