FROM alpine:3.20.3 AS builder

RUN apk add --no-cache \
    alpine-sdk \
    perl \
    git \
    openssl-dev \
    zlib-dev \
    linux-headers

RUN git clone https://github.com/wg/wrk.git \
    && cd wrk \
    && make

FROM alpine:3.20.3

RUN apk add --no-cache \
    libgcc \
    openssl \
    zlib

COPY --from=builder /wrk/wrk /usr/local/bin/

ENTRYPOINT ["wrk"]

