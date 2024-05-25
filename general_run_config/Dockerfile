FROM lukemathwalker/cargo-chef:0.1.66-rust-1 AS chef
WORKDIR /app
ARG PROGNAME

FROM chef AS planner
COPY /$PROGNAME/src  ./src
COPY /$PROGNAME/Cargo.toml  .
COPY /Cargo.lock  .
RUN cargo chef prepare --recipe-path recipe.json

FROM chef AS builder
ARG PROGNAME
COPY --from=planner /app/recipe.json recipe.json
# Build dependencies - this is the caching Docker layer!
RUN --mount=type=cache,target=/usr/local/cargo/registry \
    --mount=type=cache,target=/home/root/app/target \
    cargo chef cook --release --recipe-path recipe.json

# Build application
COPY /$PROGNAME/src  ./src
COPY /$PROGNAME/Cargo.toml  .
COPY /Cargo.lock  .
RUN --mount=type=cache,target=/usr/local/cargo/registry \
    --mount=type=cache,target=/home/root/app/target \
    cargo build --release

# We do not need the Rust toolchain to run the binary!
FROM debian:bookworm-20240423 AS runtime
WORKDIR /app
ARG PROGNAME
RUN apt-get update && apt-get install -y libssl-dev coreutils ffmpeg
# Create a script to run the command and sleep for one hour after the command is done
RUN echo "#!/bin/bash \n \
          echo \"Running command: '$PROGNAME'\" \n \
          # Run your command \n \
          $PROGNAME \n \
          echo \"Done with normal command. Sleeping for one hour\" \n \
          # Sleep for one hour \n \
          sleep 3600 \n \
          echo \"Done with sleep. Exiting\" \
          " > /app/entrypoint.sh

# Make the script executable
RUN chmod +x /app/entrypoint.sh
COPY --from=builder /app/target/release/$PROGNAME /usr/local/bin/$PROGNAME

CMD ["/app/entrypoint.sh"]
