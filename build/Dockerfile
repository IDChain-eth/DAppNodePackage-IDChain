# Build IDChain in a stock Go builder container
FROM golang:1.14-alpine as builder
RUN apk add --no-cache make gcc musl-dev linux-headers git
RUN git clone -b master https://github.com/BrightID/IDChain.git
RUN cd IDChain && make geth && cd ..

# Pull IDChain into a second stage deploy alpine container
FROM alpine:latest
COPY --from=builder /go/IDChain/build/bin/geth /usr/local/bin/geth
COPY docker-entrypoint.sh /usr/local/bin/
COPY idchain-genesis.json /idchain-genesis.json
COPY static-nodes.json /idchain/static-nodes.json
RUN apk add ca-certificates
RUN apk add \
    libstdc++ \ 
    eudev-libs \
    libgcc \
    bash

# Always execute entrypoint script
ENTRYPOINT ["docker-entrypoint.sh"]
# default parameters for entrypoint.sh
CMD ["idchain"]
