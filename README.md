# ![cosmotop](img/logo.svg)

`cosmotop` is a system monitoring tool distributed as a single executable for multiple platforms.
A fork of [`btop++`](https://github.com/aristocratos/btop) and built with
[Cosmopolitan Libc](https://github.com/jart/cosmopolitan).

## Installation

Download `cosmotop.exe` from [GitHub releases](https://github.com/bjia56/cosmotop/releases/latest).
Place it anywhere and run!

### Linux troubleshooting

Some Linux systems might be configured to launch Windows-like executables in a specific way, such as under WINE or
with WSLInterop, while others may error with a message like "run-detectors: unable to find an interpreter". In
such cases, register the file format that `cosmotop` uses with the following:

```bash
sudo wget -O /usr/bin/ape https://cosmo.zip/pub/cosmos/bin/ape-$(uname -m).elf
sudo chmod +x /usr/bin/ape
sudo sh -c "echo ':APE:M::MZqFpD::/usr/bin/ape:' >/proc/sys/fs/binfmt_misc/register"
sudo sh -c "echo ':APE-jart:M::jartsr::/usr/bin/ape:' >/proc/sys/fs/binfmt_misc/register"
```

## Usage

```
Usage: cosmotop [OPTIONS]

Options:
  -h,  --help          show this help message and exit
  -v,  --version       show version info and exit
  -lc, --low-color     disable truecolor, converts 24-bit colors to 256-color
  -t,  --tty_on        force (ON) tty mode, max 16 colors and tty friendly graph symbols
  +t,  --tty_off       force (OFF) tty mode
  -p,  --preset <id>   start with preset, integer value between 0-9
  -u,  --update <ms>   set the program update rate in milliseconds
       --utf-force     force start even if no UTF-8 locale was detected
       --debug         start in DEBUG mode: shows microsecond timer for information collect
                       and screen draw functions and sets loglevel to DEBUG
```

### Configuration

The configuration file for `cosmotop` is stored at `~/.config/cosmotop/cosmotop.conf`, populated with defaults
the first time the program runs.

### GPU monitoring

Documentation coming soon!

## Supported platforms

`cosmotop` supports the following operating systems and architectures:

- Linux 2.6.18+ (x86_64 and aarch64)
- MacOS 13+ (x86_64 and aarch64)
- Windows 10+ (x86_64)
- FreeBSD 13+ (x86_64 and aarch64)
- NetBSD 10.0+ (x86_64)
- OpenBSD 7.3 (x86_64)

## How it works

`cosmotop` uses [Cosmopolitan Libc](https://github.com/jart/cosmopolitan) and the
[Actually Portable Executable](https://justine.lol/ape.html) file format to create a single executable capable of
running on multiple operating systems and architectures. This multiplatform executable contains code to draw
the terminal UI and handle generic systems metrics, like processes, memory, disk, etc. On Windows, the executable
runs natively. On UNIX, the executable will self-extract a small loader binary to run the program.

Collecting real data from the underlying system is done by helper [plugins](https://github.com/bjia56/libcosmo_plugin), which are built for each target platform using host-native compilers and libraries. These plugins are then bundled into `cosmotop.exe` and extracted out onto the host under the path `~/.cosmotop`, and are used at runtime to gather system metrics that are then displayed by the primary multiplatform executable process in the terminal.

## Building from source

`cosmotop` is built with CMake. Both the multiplatform host executable and platform-native plugins can be built with the CMakeLists.txt at the root of this repository, but they *must be built with separate CMake invocations* due to the usage of different compilers.

### Building the multiplatform "host" executable

Download the `cosmocc` toolchain (`cosmocc-X.Y.Z.zip`) from the Cosmopolitan [GitHub releases](https://github.com/jart/cosmopolitan/releases/latest) and extract it somewhere on your filesystem. Add the `bin` directory to `PATH` to ensure the compilers can be found. For best results, compile this part on Linux.

On Linux, it may be needed to [modify `binfmt_misc`](#linux-troubleshooting) to run the `cosmocc` toolchain.

```bash
export CC=cosmocc
export CXX=cosmoc++
cmake -B build-host -DTARGET=host
cmake --build build
# or: cmake --build build --parallel
```

This should produce a `cosmotop.com` binary.

### Building platform-native "plugin" binaries

Platform-native binaries are built as shared libraries on MacOS (aarch64), Windows, and FreeBSD, and as
executables on Linux, MacOS (x86_64), NetBSD, and OpenBSD. Plugins are built as executables on platforms where Cosmopolitan
is unable to reliably load native shared libraries, or when building an executable provides greater distro compatibility (e.g. Linux plugins are statically linked with musl libc). To tell CMake to build plugins, use `-DTARGET=plugin`, which
automatically selects whether to build a shared library or an executable.

```bash
cmake -B build-plugin -DTARGET=plugin
cmake --build build
# or: cmake --build build --parallel
```

This should produce a `libcosmotop-plugin.[so|dylib]`, `cosmotop-plugin.dll`, or `cosmotop-plugin.exe`. Rename it to one of the following, matching the target platform:

```
cosmotop-linux-x86_64.exe
cosmotop-linux-aarch64.exe
cosmotop-macos-x86_64.exe
cosmotop-macos-aarch64.dylib
cosmotop-windows-x86_64.dll
cosmotop-freebsd-x86_64.so
cosmotop-freebsd-aarch64.so
cosmotop-netbsd-x86_64.exe
cosmotop-openbsd-x86_64.exe
```

### Bundling everything together

The plugin binaries can be added to `cosmotop.com` with `zip`, for example:

```bash
zip cosmotop.com cosmotop-linux-x86_64.exe
```

Themes can also be bundled:

```bash
zip -r cosmotop.com themes/
```

Optionally, rename `cosmotop.com` to `cosmotop.exe`.

## Licensing

Unless otherwise stated, the code in this repository is licensed under Apache-2.0.

`cosmotop` also relies on the following list of external libraries:
- [**Cosmopolitan Libc**](https://github.com/jart/cosmopolitan) and LLVM libcxx/compiler-rt: ISC, MIT, Apache-2.0, etc.
- [**libcosmo_plugin**](https://github.com/bjia56/libcosmo_plugin) and dependencies: MIT, Apache-2.0
- [**fmt**](https://github.com/fmtlib/fmt): MIT
- [**widecharwidth**](https://github.com/ridiculousfish/widecharwidth/): CC0-1.0
- [**range-v3**](https://github.com/ericniebler/range-v3): Boost Software License, MIT, etc.
- [**cpp-httplib**](https://github.com/yhirose/cpp-httplib): MIT