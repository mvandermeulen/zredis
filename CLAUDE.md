# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Zredis is a Zsh binary module written in C that interfaces with Redis databases via Zshell variables mapped to keys or entire databases. It allows direct manipulation of Redis data structures (hashes, lists, sets, sorted sets, strings) using native Zsh variable syntax.

## Build and Development Commands

### Building the Module

The module auto-compiles on first load, but you can manually trigger compilation:

```zsh
# Manual compilation
zredis_compile

# Or directly via script
./zredis_compile
```

The build process:
- Requires `hiredis` library (system dependency)
- Uses autoconf/configure build system
- Build configuration via zstyles (see ZSTYLES.md)
- Outputs modules to `module/Src/zshell/` (zredis.so, db.so, zgdbm.so)
- Creates timestamp in `module/COMPILED_AT`

Build customization via zstyles (set before loading plugin):
```zsh
zstyle ":plugin:zredis" cppflags "-I/usr/local/include"
zstyle ":plugin:zredis" cflags "-Wall -O2 -g"
zstyle ":plugin:zredis" ldflags "-L/usr/local/lib"
zstyle ":plugin:zredis" configure_opts ""  # Additional ./configure options
```

### Running Tests

```zsh
# Run Valgrind test suite (VATS)
cd module/VATS
./runtests.zsh

# Configuration in module/VATS/vtest.conf:
# - test_bin: Binary to test (default: "local-zsh" = ../Src/zsh)
# - zsh_control_bin: Binary for test orchestration (default: "zsh")
# - tkind: Test kind - "error", "leak", or "nopossiblylost"
```

Test files are in `module/VATS/V*.ztst` format. Tests require a running Redis server (configured via `module/VATS/z-redis.conf`).

## Architecture

### Module Structure

- **module/Src/zshell/zredis.c** - Main Redis bindings implementation
- **module/Src/zshell/zgdbm.c** - GDBM database bindings
- **module/Src/zshell/db.c** - Common database interface layer
- **module/Src/zshell/db.h** - Database backend definitions and types
- **module/Src/** - Zsh core utilities (from upstream Zsh source)

### Key Components

**Database Backend Commands** (db.h):
- DB_TIE (1) - Bind variable to database
- DB_UNTIE (2) - Unbind variable
- DB_IS_TIED (3) - Check if variable is tied
- DB_GET_ADDRESS (4) - Get database address
- DB_CLEAR_CACHE (5) - Clear read cache

**Redis Type Mappings**:
- DB_KEY_TYPE_STRING (3) → Zsh string
- DB_KEY_TYPE_LIST (4) → Zsh array
- DB_KEY_TYPE_SET (5) → Zsh array
- DB_KEY_TYPE_ZSET (6) → Zsh hash (member → score)
- DB_KEY_TYPE_HASH (7) → Zsh hash
- DB_KEY_TYPE_NO_KEY (2) → Whole database as hash

### Core Mechanisms

**Variable Binding (`ztie` command)**:
```zsh
ztie -d db/redis -a "HOST:PORT/DB_INDEX/KEY" variable_name
```

Options:
- `-r` - Read-only
- `-z` - Zero-cache (disable read caching)
- `-L {type}` - Lazy binding (create key on write)
- `-D` - Delete key on unset
- `-S` - Super-lazy (defer connection until first use)

**GSU (Get/Set/Unset) Extensions**:
- `gsu_scalar_ext` - For hash elements and strings, includes Redis connection context
- `gsu_array_ext` - For sets/lists, includes Redis connection context
- Each GSU carries: type, cache flags, Redis host/port, key, password, connection handle

**Caching System**:
- Reads are cached by default (first read hits DB, subsequent reads use cache)
- Writes are never cached
- Clear cache: `ztclear variable_name [key]`
- Disable cache: use `-z` flag with `ztie`

### Plugin Loading

The `zredis.plugin.zsh` script:
1. Sets up `ZREDIS_REPO_DIR` and `ZREDIS_CONFIG_DIR`
2. Updates `fpath` if not using a plugin manager
3. Checks if module needs compilation (via `module/RECOMPILE_REQUEST` timestamp)
4. Compiles module if needed (thread-safe with flock)
5. Loads module via `zmodload zshell/zredis`

## Redis Connection Format

Host specification: `HOST:PORT/DB_INDEX[/KEY]`
- HOST - Redis server hostname/IP (default: 127.0.0.1)
- PORT - Redis port (default: 6379)
- DB_INDEX - Database number (0-15 typically)
- KEY - Optional specific key to bind (omit for whole-database mapping)

## Development Notes

- Module auto-recompiles when `module/RECOMPILE_REQUEST` is newer than `module/COMPILED_AT`
- Multiple shells can load simultaneously; first one compiles (uses flock)
- Zsh version detection via `is-at-least zsh-5.6.1-dev-1` sets compile flag `ZREDIS_ZSH_262_DEV_1`
- Core Zsh utilities in `module/Src/` are copied from upstream Zsh (see `copy_from_zsh_src.zsh`)
