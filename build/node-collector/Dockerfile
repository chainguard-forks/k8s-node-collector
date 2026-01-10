FROM alpine:20230901@sha256:f2d1645cd73c7e54584dc225da0b5229d19223412d719669ebda764f41396853
RUN apk update && apk add --upgrade libcrypto3 libssl3

COPY node-collector /usr/local/bin/node-collector

ENTRYPOINT ["node-collector"]