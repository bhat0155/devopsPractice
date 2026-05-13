# syntax=docker/dockerfile:1
ARG RUST_VERSION=1.85.0
ARG APP_NAME=shipping

################################################################################
# Create a stage for building the application.
FROM rust:${RUST_VERSION}-alpine AS build
ARG APP_NAME
WORKDIR /app

# Install host build dependencies.
RUN apk add --no-cache clang lld musl-dev git protobuf-dev

# Copy proto file to expected location for dockerproto feature
COPY pb/demo.proto /app/proto/demo.proto

# Build the application.
RUN --mount=type=bind,source=src/shipping/src,target=src \
    --mount=type=bind,source=src/shipping/Cargo.toml,target=Cargo.toml \
    --mount=type=bind,source=src/shipping/Cargo.lock,target=Cargo.lock \
    --mount=type=bind,source=src/shipping/build.rs,target=build.rs \
    --mount=type=cache,target=/app/target/ \
    --mount=type=cache,target=/usr/local/cargo/git/db \
    --mount=type=cache,target=/usr/local/cargo/registry/ \
    cargo build --locked --release --features dockerproto && \
    cp ./target/release/$APP_NAME /bin/server

################################################################################
FROM alpine:3.18 AS final

ARG UID=10001
RUN adduser \
    --disabled-password \
    --gecos "" \
    --home "/nonexistent" \
    --shell "/sbin/nologin" \
    --no-create-home \
    --uid "${UID}" \
    appuser

USER appuser

COPY --from=build /bin/server /bin/

EXPOSE 7077

CMD ["/bin/server"]

