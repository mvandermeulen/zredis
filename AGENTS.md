# Repository Guidelines

## Project Structure & Module Organization
The repository root holds the plugin entrypoint `zredis.plugin.zsh` alongside the rebuild helper `zredis_compile`. Core C sources live in `module/Src`, with Redis bindings implemented in `module/Src/zshell/zredis.c` and Autotools metadata close by under `module`. Documentation stays in `docs/`, while `.ztst` suites and fixtures under `module/Test` mirror the upstream zsh layout and group scenarios by prefix.

## Build, Test, and Development Commands
Run `./zredis_compile` to configure, build, and timestamp the module using the default zstyle-driven flags; it refreshes `module/Src/zshell/{zredis,zgdbm}.c` and calls `make`. For scripted or manual flows, execute `cd module && ./configure --enable-gdbm && make`, setting `CPPFLAGS`, `CFLAGS`, or `LDFLAGS` when `hiredis` lives outside `/usr/local`. Use `make check` (or `ZTST_verbose=1 make check`) inside `module` for the full suite, and shorten cycles with `make TESTNUM=V12 check` to run only the zredis cases.

## Coding Style & Naming Conventions
Honor `module/.editorconfig` and `module/Src/.indent.pro`: LF endings, four-space indentation for C, linux defaults, and no hard tabs. Keep identifiers snake_case (`zredis_tied`, `zrupdate_zredis_last`) and reuse existing helpers instead of introducing near duplicates. Shell helpers such as `module/VATS/runtests.zsh` expect two-space indentation with spaces only; keep comments brief and task-oriented.

## Testing Guidelines
`.ztst` files are grouped by subsystem (A for parsing, B for builtins, V for modules, etc.), with Redis behaviour covered in `V12zredis.ztst`–`V17zredis_list.ztst`. Base new tests on the chunk format explained in `module/Test/B01cd.ztst` so indentation and expected-output blocks parse correctly. Always run `make check` before opening a pull request and include verbose logs when reporting or triaging failures.

## Commit & Pull Request Guidelines
Recent history follows Conventional Commits (`build(deps): …`, `fix: …`), so match that style and keep each commit focused on a single logical change. Pull requests should explain motivation, outline functional impact, and note dependency adjustments; attach CLI transcript snippets or screenshots when behaviour changes. Confirm executable bits on scripts such as `zredis_compile` and call out new environment requirements directly in the PR body.

## Security & Configuration Tips
Follow `docs/SECURITY.md` for coordinated disclosure and report issues privately before publishing. Prefer zstyles (e.g., the `:plugin:zredis` namespace) to expose configurable flags rather than modifying tracked files. Keep Redis credentials and machine-specific configuration out of version control by relying on environment variables or ignored files during development.
