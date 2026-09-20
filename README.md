# tree-sitter CLI releases for glibc 2.17

[![Release](https://github.com/arusuki/tree-sitter-releases/actions/workflows/release.yml/badge.svg)](https://github.com/arusuki/tree-sitter-releases/actions/workflows/release.yml)

Unofficial, best-effort Linux builds of the upstream
[`tree-sitter` CLI](https://github.com/tree-sitter/tree-sitter), linked for a
glibc 2.17 baseline.

These builds are intended for older Linux distributions that cannot run the
official binaries. They are not maintained or supported by the tree-sitter
project.

## Release assets

| Platform | Rust target | Assets |
| --- | --- | --- |
| Linux x86-64 | `x86_64-unknown-linux-gnu` | `.gz`, `.zip` |
| Linux ARM64 | `aarch64-unknown-linux-gnu` | `.gz`, `.zip` |

Every release also includes `SHA256SUMS` and GitHub build-provenance
attestations. The executable inside each archive is named `tree-sitter`.

## How it works

The [release workflow](.github/workflows/release.yml):

1. Checks the latest stable upstream release on the first day of each month.
2. Checks out the exact upstream tag. Where needed, translates the Linux allocator
   export linker option from `--dynamic-list` to `--export-dynamic` plus an anonymous
   version script with the same four exports. No parser or CLI source is changed;
   CI fails if the expected upstream build-script layout changes.
3. Builds `tree-sitter-cli` with Rust and Zig via `cargo-zigbuild`, targeting glibc 2.17.
4. Runs the binary and inspects its ELF symbol versions, failing if any required
   `GLIBC_*` version is newer than 2.17. Also runs `--version` and `build --help`
   inside a manylinux2014 container whose libc version is asserted to be 2.17.
   Checks the allocator exports used by external scanners when upstream requests them.
5. Publishes compressed binaries, checksums, and provenance to a release with
   the same tag.

If that upstream tag already has a release here, the scheduled run exits
without rebuilding it.

## Manual build

Open **Actions → Release → Run workflow** and use one of these values:

- `latest` — build the latest stable upstream release.
- An exact tag such as `v0.28.0` — build that upstream tag.

Enable `force_rebuild` to replace assets on an existing release without
deleting the release.

## Install

Download the archive for your architecture from
[Releases](https://github.com/arusuki/tree-sitter-releases/releases), verify it
against `SHA256SUMS`, then place the executable on your `PATH`. For example:

```bash
gzip -dc tree-sitter-linux-x64-glibc-2.17.gz > tree-sitter
chmod +x tree-sitter
./tree-sitter --version
```

The glibc guarantee applies to the CLI executable itself. Parsers or external
scanners compiled later must also be built for the target system's ABI.

## Build inputs

- Source: [`tree-sitter/tree-sitter`](https://github.com/tree-sitter/tree-sitter)
- glibc baseline: `2.17`
- Zig: `0.16.0` (downloaded from ziglang.org and SHA-256 verified)
- cargo-zigbuild: `0.23.4` (upstream release binary, SHA-256 verified)
- Rust dependencies: the upstream tag's committed `Cargo.lock`
