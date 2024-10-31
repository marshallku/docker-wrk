# Build stage
FROM alpine:3.18 AS builder

# Install build dependencies including linux headers
RUN apk add --no-cache \
    alpine-sdk \
    perl \
    git \
    openssl-dev \
    zlib-dev \
    linux-headers

# Clone and build wrk
RUN git clone https://github.com/wg/wrk.git \
    && cd wrk \
    && make

# Final stage
FROM alpine:3.18

# Install runtime dependencies only
RUN apk add --no-cache \
    libgcc \
    openssl \
    zlib

# Copy wrk binary from builder
COPY --from=builder /wrk/wrk /usr/local/bin/

# Set the entrypoint
ENTRYPOINT ["wrk"]

