# Scanner image for https://github.com/tari-project/tari (development).
#
# The audit runs offline, so this build fetches every crate in Cargo.lock, then builds the workspace in the dev profile
# (debug assertions and overflow checks on): the libraries, the binaries (minotari_node, minotari_console_wallet,
# minotari_miner, minotari_merge_mining_proxy, ...) and every test binary. threat_model.md, next to this file, says how
# to run them.

# rust-toolchain.toml pins the stable channel, which is what CI builds with.
FROM rust:1-bookworm
ENV DEBIAN_FRONTEND=noninteractive \
    RUST_BACKTRACE=1 \
    CARGO_TERM_COLOR=never

# scanner contract: the checkout lives inside the image, at /src
COPY . /src
WORKDIR /src

# The system dependencies CI installs (sqlite, openssl, hidapi/udev for the Ledger transport, and an upstream protoc
# new enough for proto3 `optional`).
RUN apt-get update \
    && bash scripts/install_ubuntu_dependencies.sh \
    && rm -rf /var/lib/apt/lists/*

# Every crate in Cargo.lock, so later builds with other features or cfgs stay offline.
RUN cargo fetch --locked

# The workspace: libraries, binaries and every test binary. A step that fails fails the build, and primary_contact
# gets the log.
RUN cargo build --locked --workspace --lib --bins \
    && cargo test --locked --workspace --no-run

# Run the unit tests so the log shows the image works, without letting a failure abort the build. Some tari_core tests
# share state and flake when run in parallel, so run them on one thread. The cucumber suite (integration_tests) spins
# up whole networks and is not run here.
RUN cargo test --locked --workspace --lib --exclude tari_integration_tests -- --test-threads=1 \
    || echo "WARNING: cargo test reported failures"
