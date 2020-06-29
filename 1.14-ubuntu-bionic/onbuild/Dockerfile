FROM markusma/golang:1.14-ubuntu-bionic

RUN mkdir -p /go/src/app
WORKDIR /go/src/app

# this will ideally be built by the ONBUILD below ;)
CMD ["go", "run"]

ONBUILD COPY . /go/src/app
ONBUILD RUN go download
ONBUILD RUN go install
