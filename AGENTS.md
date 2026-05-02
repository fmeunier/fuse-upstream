# AGENTS.md — Guidance for AI Coding Assistants

This file provides orientation for AI coding agents (Copilot, Claude, etc.)
working on the **Fuse ZX Spectrum Emulator** codebase.

## Project Overview

Fuse is an accurate, portable emulator of the Sinclair ZX Spectrum (and
clones), written in C. The primary platforms are Linux, macOS, and Windows.
Wii is also supported as a cross-compilation target.

## Build System

Fuse uses **Autotools** (autoconf + automake):

```sh
./autogen.sh          # Generate configure script (requires autoconf, automake)
./configure           # Configure for the host platform
make                  # Build
sudo make install     # Install
make check            # Run unit tests
```

A `libspectrum` library (from `arki55/fuse-libspectrum`) must be installed
before building Fuse. See `INSTALL` for details.

## Coding Style

See `hacking/coding_style.txt` for the full style guide. Key points:

- **Language**: C (not C++).
- **Indentation**: 2 spaces (no tabs).
- **Naming**: `snake_case` for variables and functions;
  `UPPER_SNAKE_CASE` for `#define` and `enum` constants.
- **Spaces**: add spaces after `(` and before `)` in function calls when
  arguments are present, and around binary operators.
- **Braces**: opening brace on the same line as the control construct
  (except function definitions); cuddled `} else {`.
- **New source files**: must follow the style guide; do not reformat
  existing code that deviates from it.

## Repository Layout

| Path | Contents |
|------|----------|
| `fuse.c` | Entry point and main emulation loop |
| `z80/` | Z80 CPU emulation core |
| `machines/` | Machine-specific configurations (48K, 128K, +2, +3, …) |
| `peripherals/` | Peripheral emulation (Interface 1, Kempston, Multiface, …) |
| `ui/` | User-interface backends (GTK3, SDL, null, …) |
| `sound/` | Audio backends (OSS, SDL, CoreAudio, …) |
| `display.c` | Screen image builder |
| `event.c` | Tstate-based event scheduler |
| `tape.c` | Tape loading/saving |
| `snapshot.c` | Snapshot (`.z80`, `.sna`, …) support |
| `memory_pages.c` | Memory paging logic |
| `unittests/` | Unit test suite |
| `hacking/` | Developer documentation and notes |
| `data/` | Desktop integration files (AppStream, .desktop) |
| `.github/workflows/` | CI pipelines (Linux, macOS, Windows, Wii) |

## CI / GitHub Actions

Pull requests trigger builds for Linux (multiple UIs), macOS, Windows
(Cygwin/MinGW), and Wii. The workflows are in `.github/workflows/`.

Key notes:
- The `libspectrum` library is built by reusable workflows in this
  repository, using source fetched from the external libspectrum
  repository, and then passed to the Fuse build jobs as an artifact.
- Keep the `upload-artifact` and `download-artifact` versions in the
  libspectrum and Fuse workflows aligned with the versions currently in
  use by this repository.

## Contribution Guidelines

- **Do not introduce new dependencies** without prior discussion in an issue.
- **No breaking changes** without maintainer approval.
- **Small, focused patches** — one concern per commit/PR.
- Match the existing coding style of the file you are editing.
- If editing generated files (`menu_data.dat`, `settings.dat`, `keysyms.dat`),
  also regenerate the corresponding C sources with the provided Perl scripts
  (`.pl` files).
- Run `make check` before submitting a PR.

## Key Concepts

- **tstates**: The Z80 clock cycle count since the last interrupt. Fuse's
  event scheduler fires when `tstates` reaches `event_next_event`.
- **ULA**: The Uncommitted Logic Array — the custom chip handling screen,
  keyboard, and tape I/O on real Spectrum hardware.
- **Modules**: Fuse uses a simple module system (`module.c`) to allow
  components to register reset/snapshot/config callbacks.
