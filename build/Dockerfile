# Multi-stage Docker build for OpenTelemetry Collector

# Stage 1: Get certificates
FROM alpine:3.20 AS certs
RUN apk --update add ca-certificates

# Stage 2: Build the collector
FROM golang:1.24-alpine AS build-stage

# Install build dependencies
RUN apk add --no-cache git

WORKDIR /build

# Copy builder configuration
COPY builder-config.yaml .

# Copy local component repositories (context is build directory, so we copy from parent)
COPY ../Dynatrace-opentelemetry-collector-contrib ./Dynatrace-opentelemetry-collector-contrib
COPY ../opentelemetry-collector-contrib ./opentelemetry-collector-contrib

# Install OCB (OpenTelemetry Collector Builder)
RUN --mount=type=cache,target=/root/.cache/go-build \
    --mount=type=cache,target=/go/pkg/mod \
    GO111MODULE=on go install go.opentelemetry.io/collector/cmd/builder@v0.137.0

# Build the custom collector
RUN --mount=type=cache,target=/root/.cache/go-build \
    --mount=type=cache,target=/go/pkg/mod \
    builder --config builder-config.yaml

# Stage 3: Runtime image
FROM gcr.io/distroless/static:latest

# Set up user
ARG USER_UID=10001
USER ${USER_UID}

# Copy certificates
COPY --from=certs /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/ca-certificates.crt

# Copy collector configuration
COPY collector-config.yaml /etc/otelcol/config.yaml

# Copy the built collector binary
COPY --from=build-stage --chmod=755 /build/otelcol-dynatrace/otelcol-dynatrace /otelcol/otelcol-dynatrace

# Set entrypoint
ENTRYPOINT ["/otelcol/otelcol-dynatrace"]
CMD ["--config", "/etc/otelcol/config.yaml"]

# Expose standard OpenTelemetry ports
EXPOSE 4317 4318 8888

# Add labels
LABEL org.opencontainers.image.title="Dynatrace OpenTelemetry Collector"
LABEL org.opencontainers.image.description="Custom OpenTelemetry Collector with Dynatrace components"
LABEL org.opencontainers.image.source="https://github.com/solarekm/observability"