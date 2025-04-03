FROM golang:latest AS builder
WORKDIR /app

ARG DERP_VERSION=latest
RUN go install tailscale.com/cmd/derper@${DERP_VERSION}

FROM ubuntu:22.04
WORKDIR /app
ADD entrypoint.sh /app

RUN chmod +x /app/entrypoint.sh

ENV DERP_DOMAIN=your-hostname.com
ENV DERP_ADDR=:8080
ENV DERP_HTTP_PORT=8080

COPY --from=builder /go/bin/derper .

ENTRYPOINT ["/app/entrypoint.sh"]
