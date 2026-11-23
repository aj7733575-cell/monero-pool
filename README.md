# monero-pool

Running a Monero mining pool requires a server that accepts miner connections, validates shares, and processes payouts. monero-pool is a Monero mining pool server written in C, with libevent for client connections and LMDB for storage.

## Features

- Accept Stratum miner connections through libevent, with separate threads for Stratum clients and web UI clients under normal operation.
- Process payouts using PPLNS, the pool's only payout mechanism.
- Support RandomX mining, including fast/full-memory mode.
- Let miners choose their own block templates through the optional self-select Stratum mode.
- Connect edge pools to an upstream pool that processes payouts and receives validated shares and blocks. Edge pools retain a backlog while the upstream is unreachable.
- Fetch new block templates on SIGUSR1 when block notification mode is enabled, instead of polling for new blocks.

## Installation

First, clone and compile Monero according to the [Monero build instructions](https://github.com/monero-project/monero#compiling-monero-from-source). The pool build needs that compiled source tree.

On Ubuntu, install the additional pool dependencies:

```bash
sudo apt-get install liblmdb-dev libevent-dev libjson-c-dev uuid-dev
```

From the monero-pool source directory, point the build at your Monero checkout and compile:

```bash
export MONERO_ROOT=/path/to/cloned/monero
make release
```

The release build is written to `build/release/`. Run `make` instead for a debug build in `build/debug/`.

For a release-archive download, replace `OWNER` below with this standalone project's GitHub owner. This command assumes a `monero-pool-v0.1.0.tar.gz` asset has been published on its GitHub Releases page; the supplied project data does not confirm that release exists.

```bash
curl -fL -o monero-pool-v0.1.0.tar.gz https://github.com/OWNER/monero-pool/releases/download/v0.1.0/monero-pool-v0.1.0.tar.gz
```

## Usage

The build copies `pool.conf` into the output directory. Edit that file for your deployment before starting the pool, then pass its location explicitly:

```bash
./build/release/monero-pool --config-file ./build/release/pool.conf
```

Without `--config-file`, the pool checks for `pool.conf` beside the executable, then in the current user's home directory.

To use daemon-triggered block notifications, start the pool with:

```bash
./build/release/monero-pool --config-file ./build/release/pool.conf --block-notified
```

Add the following option to your usual `monerod` command:

```bash
--block-notify '/usr/bin/pkill -USR1 monero-pool'
```

This sends SIGUSR1 to matching pool processes whenever the daemon adds a block to the chain. In this mode, the pool relies on those signals rather than its polling timer.

For interconnected deployments, configure `trusted-listen`, `trusted-port`, and `trusted-allowed` on the upstream instance. Set `upstream-host` and `upstream-port` on each downstream instance. **Keep the trusted listener private:** downstream pools perform share validation, so access to this listener must be restricted to trusted instances.

## License

Licensed under BSD-3-Clause. This project is based on [jtgrassie/monero-pool](https://github.com/jtgrassie/monero-pool). Retain the upstream copyright notice, license conditions, and disclaimer when redistributing the source; include them in the accompanying materials for binary distributions.

## Related projects

Other repositories supplied as projects from the same author; their implementations were not included in the source material:

- [Cabral-it/docker-ikev2-vpn-server](https://github.com/Cabral-it/docker-ikev2-vpn-server) — named for a Docker-based IKEv2 VPN server.
- [ghb13/node-hnsn](https://github.com/ghb13/node-hnsn) — no functional description was supplied; consult its README for its purpose.
- [houdinetsounds/claude-config-editor-eqhg](https://github.com/houdinetsounds/claude-config-editor-eqhg) — named for a Claude configuration editor.

