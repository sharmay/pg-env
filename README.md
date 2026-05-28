# pg-env

A lightweight shell environment manager for PostgreSQL development from source. Activates a virtual-environment-style shell session with isolated paths, ports, and helper functions for building, testing, and managing multiple PostgreSQL instances simultaneously.

Automatically detects whether the source tree uses **meson** (PG 16+) or **autoconf** and adjusts all build commands accordingly.

## Requirements

- zsh or bash 4+
- Git (with worktree support)
- PostgreSQL build dependencies (gcc/clang, make, meson/ninja, libreadline, zlib, openssl, lz4, zstd, perl, python, etc.)
- `prove` (from perl Test::Harness) for TAP tests

## Setup

```bash
# Clone pg-env and symlink into PATH
git clone <repo> ~/pg-env
ln -s ~/pg-env/pg-env ~/bin/pg-env
```

## Quick Start

```bash
# 1. Clone PostgreSQL and create a version worktree
cd ~/gitwork
git clone https://git.postgresql.org/git/postgresql.git
git -C postgresql worktree add --track -b REL_18_STABLE ../postgresql-18 origin/REL_18_STABLE
mkdir -p postgresql-18-INST

# 2. Activate pg-env
export PGV=18
source ~/bin/pg-env

# 3. Build and run
configure && compile && install
compile_contrib
setupdb
startdb
psql
```

Your prompt changes to `(PG:18)` indicating the active environment.

## How It Works

When sourced, `pg-env` sets up environment variables (`PATH`, `PGDATA`, `PGPORT`, etc.) pointing to a version-specific source tree and install directory, then defines shell functions for common development tasks. When done, `deactivate` restores your original environment.

### Directory Layout

```
~/gitwork/                       ($GIT_DIR)
  postgresql/                    main git clone
  postgresql-18/                 worktree for PG 18 ($PG_DEV_SRC)
  postgresql-18-INST/            install prefix ($PG_DEV_INST)
    bin/                         pg_config, psql, etc.
    data/                        default PGDATA
    data-1/, data-2/, data-3/    multi-node data directories
    lib/                         shared libraries
  postgresql-HEAD/               worktree for development HEAD
  postgresql-HEAD-INST/          install prefix for HEAD
  pgconfigs/                     shared postgresql.conf includes
    pg18.conf                    default node config
    pg18-1.conf, pg18-2.conf     per-node configs
```

### Port Assignment

| PGV | Default Port | Node 1 | Node 2 | Node 3 |
|-----|-------------|--------|--------|--------|
| 18 | 5418 | 15418 | 25418 | 35418 |
| 17 | 5417 | 15417 | 25417 | 35417 |
| HEAD | 5432 | 15432 | 25432 | 35432 |

- Numeric versions: port = `54<version>`
- Non-numeric versions (HEAD): port = 5432
- Multi-node: node N uses port `N<PGPORT>` (max 5 nodes)

### PostgreSQL Configuration Files

Global configs are stored in `${GIT_DIR}/pgconfigs/` with this naming convention:

```
pg${PGV}.conf              # default instance
pg${PGV}-<N>.conf          # node N
```

Per-project local configs (stored in the working directory) overlay on top of global configs. They use the same naming but prefixed with a dot:

```
.pg${PGV}.conf             # default instance
.pg${PGV}-<N>.conf         # node N
```

Both are included via `postgresql.conf` `include` directives during `setupdb`.

## Configuration

pg-env loads configuration from two optional files (in order):

1. `~/.pg-env` - global defaults
2. `.pg-env` - per-project overrides (in current working directory)

### Core Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `PGV` | (required) | PostgreSQL version (e.g. `18`, `HEAD`) |
| `GIT_DIR` | `~/gitwork` | Base directory for source/install trees |
| `PG_DEV_NAME` | `postgresql-${PGV}` | Source directory name |
| `PG_INST_NAME` | `${PG_DEV_NAME}-INST` | Install directory name |
| `PG_DEV_SRC` | `${GIT_DIR}/${PG_DEV_NAME}` | Full path to source tree |
| `SKIP_MESON_BUILD` | (unset) | Set to `1` to force autoconf even if `meson.build` exists |
| `MAKEFLAGS` | (unset) | Make settings, e.g. `-j 4` |

### Build Flags

| Variable | Description |
|----------|-------------|
| `PG_ENV_CFLAGS` | Extra CFLAGS appended to all builds |
| `PG_ENV_CONFIGURE_FLAGS` | Extra `./configure` flags for all versions |
| `PG_ENV_CONFIGURE_FLAGS_<V>` | Version-specific configure flags (overrides generic) |
| `PG_ENV_MESON_FLAGS` | Extra `meson setup` flags for all versions |
| `PG_ENV_MESON_FLAGS_<V>` | Version-specific meson flags (overrides generic) |

The version-specific variable takes precedence over the generic one. `<V>` is the uppercased PGV value (e.g. `HEAD`, `18`).

### Debugging

| Variable | Description |
|----------|-------------|
| `PG_ENV_DEBUG` | Set to any non-empty value to print debug info during sourcing |

### Example `~/.pg-env`

```bash
GIT_DIR=${HOME}/gitwork

# Enable address sanitizer for all builds
PG_ENV_CFLAGS="-fsanitize=address"

# HEAD-only flags
PG_ENV_CONFIGURE_FLAGS_HEAD="--enable-injection-points"
PG_ENV_MESON_FLAGS_HEAD="-Dinjection_points=true"

# PG 18 specific
PG_ENV_CONFIGURE_FLAGS_18="--with-icu"
```

### Example per-project `.pg-env`

```bash
PGV=18
PG_DEV_NAME=postgresql-CF_1234_18
#SKIP_MESON_BUILD=1
```

## Commands Reference

Run `help` after activating pg-env for a context-aware summary showing whether autoconf or meson is active.

### Info

| Command | Description |
|---------|-------------|
| `help` | Show all available commands (adapts to current build system) |
| `penv` | Print current environment variables (PGV, paths, ports, flags) |
| `status` | Show running/stopped PostgreSQL instances with PIDs and ports |
| `pglog [N]` | Tail the latest log file (default instance, or node N) |

### Build

Commands adapt automatically to the detected build system (autoconf/make or meson/ninja).

| Command | Autoconf | Meson |
|---------|----------|-------|
| `configure` | `./configure ...` | `meson setup build ...` |
| `compile` | `make -j` | `ninja` |
| `install` | `make -j install` | `ninja install` |
| `compile_contrib` | `make -j install` (in contrib/) | (included in main build) |
| `distclean` | `make distclean` | `ninja clean` |
| `docs` | `make docs` | `ninja docs` |

Typical workflows:

```bash
# Full initial build
configure && compile && install && compile_contrib

# Incremental rebuild after code changes
compile && install

# Rebuild from scratch
resetinst
```

### Database Lifecycle

All commands accept optional node numbers. No arguments operates on the default instance.

| Command | Description |
|---------|-------------|
| `setupdb [N ...]` | Run `initdb` and configure `postgresql.conf` includes |
| `startdb [N ...]` | Start instance(s) via `pg_ctl` |
| `stopdb [N ...]` | Stop instance(s) via `pg_ctl` |
| `restartdb [N ...]` | Stop then start instance(s) |

```bash
# Single instance workflow
setupdb && startdb

# Multi-node workflow
setupdb 1 2 3
startdb 1 2 3
stopdb 2            # stop just node 2
restartdb 1 3       # restart nodes 1 and 3
```

### Cleanup and Reset

| Command | Description |
|---------|-------------|
| `cleandb [N ...]` | Stop and remove data directory contents |
| `cleaninst` | Remove install directory contents |
| `cleanall [N ...]` | `cleandb` + `cleaninst` |
| `resetdb [N ...]` | `cleandb` + `setupdb` + `startdb` |
| `resetinst` | `distclean` + `configure` + `compile` + `install` + `compile_contrib` |
| `resetall [N ...]` | `resetinst` + `resetdb` |

### Query and Test

| Command | Autoconf | Meson |
|---------|----------|-------|
| `c` | `make check` | `meson test --suite setup --suite regress` |
| `cw` | `make check-world` | `meson test` (all suites) |
| `ic` | `make installcheck` | `meson test --setup running --suite regress-running` |
| `icw` | `make installcheck-world` | `meson test --setup running` |

| Command | Description |
|---------|-------------|
| `pexec [N ...] [PEXEC_CMD="..."]` | Run psql on default or specified nodes |
| `r <test> [...]` | Run specific SQL regression test(s) by name |
| `t <test.pl> [...]` | Run specific TAP/Perl test(s) by path |

```bash
# SQL regression tests
r int4 int8 float4

# TAP tests (from source root)
t src/test/recovery/t/001_stream_rep.pl

# TAP test from within the test directory
cd src/test/recovery
t t/001_stream_rep.pl

# Execute SQL on multiple nodes
pexec 1 2 3 PEXEC_CMD="-c 'SELECT pg_is_in_recovery()'"

# Interactive psql on node 2
pexec 2

# Run installcheck after code change
compile && install && ic
```

### Exit

| Command | Description |
|---------|-------------|
| `deactivate` | Stop all instances (default + nodes 1-3) and restore original shell environment |

## Standalone Commands

When executed directly (not sourced), `pg-env` provides commands for managing PostgreSQL worktrees. Run these from within a PostgreSQL git checkout directory.

### Create a Development Branch

```bash
# Branch from current HEAD
pg-env branch my-feature

# Branch from a released version
pg-env branch my-feature REL_18_STABLE
```

This creates a git worktree at `../postgresql-<branch_name>` with a `.pg-env` config file ready for sourcing.

### Test a Commitfest Patch

Visit https://commitfest.postgresql.org/, click on an active commitfest, then click on an entry. The Entry ID is in the URL (e.g. `https://commitfest.postgresql.org/48/4962/` -> Entry ID is `4962`).

```bash
# Apply a patch from commitfest against HEAD
pg-env commitfest 4962

# Apply against a specific version (e.g. PG 18)
pg-env commitfest 4962 18
```

This creates a worktree, downloads the latest patch attachment from the commitfest page, and applies it with `git am`. You can then `source pg-env` from the new worktree directory to build and test.

## Multi-Version Workflow

You can work with multiple PostgreSQL versions in separate terminal sessions:

```bash
# Terminal 1: PG 18
export PGV=18
source ~/bin/pg-env
configure && compile && install
setupdb && startdb

# Terminal 2: PG HEAD
export PGV=HEAD
source ~/bin/pg-env
configure && compile && install
setupdb && startdb
```

Each session has isolated `PATH`, `PGDATA`, `PGPORT`, and all helper functions operate on their respective version.

## License

Copyright 2018-2024 Yogesh Sharma
