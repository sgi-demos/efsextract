# efsextract

Extract files from SGI CD images or EFS file systems.

Silicon Graphics (SGI) shipped much of its software on CDs that are not ISO 9660
discs but raw hard-disk images pressed to CD, complete with an SGI disk label and
an EFS partition. Most systems can't read them. `efsextract` lets non-SGI systems
list and extract the files from such discs (or from a bare EFS file system).

## Usage

```
efsextract [OPTION] [FILE]
efsextract [-h|-V]
```

| Option       | Description                                                        |
|--------------|--------------------------------------------------------------------|
| `-h`         | Print a usage message and exit successfully.                       |
| `-l`         | List files without extracting.                                     |
| `-L`         | List partitions and bootfiles from the volume header.              |
| `-o ARCHIVE` | Instead of extracting, create a tar archive containing all files.  |
| `-p NUM`     | Use partition number `NUM` (default: 7).                           |
| `-q`         | Do not show the file listing while extracting.                     |
| `-V`         | Print version information and exit successfully.                   |
| `-W`         | Scan the image for `inst` packages and list them.                  |
| `-X`         | Extract bootfiles from the volume header.                          |

On Windows, fifos, device nodes, sockets, and symlinks can't be recreated, so
those entries are skipped with a warning during extraction.

## Building

The only dependency is **libcdio** (specifically `libiso9660` and `libcdio`).
There are separate makefiles per platform; pick the one that matches your host.

### Windows — MSYS2 CLANG64 (native build)

This is the simplest way to get a native Windows binary. From an **MSYS2 CLANG64**
shell:

```sh
# 1. Install the toolchain and libcdio (once)
pacman -S --needed make \
                   mingw-w64-clang-x86_64-clang \
                   mingw-w64-clang-x86_64-libcdio

# 2. Build
make -f Makefile.clang64
```

This produces `efsextract.exe`, linked dynamically against the libcdio DLLs. It
runs as-is inside the CLANG64 shell because `/clang64/bin` is on your `PATH`.

To produce a **standalone** executable that runs without the MSYS2 DLLs:

```sh
make -f Makefile.clang64 STATIC=1
```

If static linking reports undefined references, add the libraries it names (most
commonly `-lwinmm` or `-liconv`) to the `STATIC` block in `Makefile.clang64`.

Notes:

- libcdio comes from pacman and installs into the CLANG64 sysroot, so no
  `-I`/`-L` flags or `pkg-config` calls are needed.
- `-lws2_32` is linked because the portable endian helpers map the 64-bit byte
  swaps onto winsock's `htonll`/`ntohll`.

### Windows — cross-compile from Linux

`Makefile.win` builds a 32-bit, statically linked, UPX-compressed `.exe` using the
MinGW-w64 cross toolchain. It builds libcdio from the bundled tarball in
`libraries/`, so you don't need a system libcdio — just the cross compiler,
`upx`, and `strip`.

```sh
make -f Makefile.win
```

Adjust the `HOST` variable (default `i686-w64-mingw32`) for a different target
triplet.

### Linux / Unix

Install libcdio's development files, then build with the default makefile.

```sh
# Debian / Ubuntu
sudo apt install libcdio-dev libiso9660-dev

# Fedora
sudo dnf install libcdio-devel

make
sudo make install      # installs to /usr/local
```

`make install` places the binary in `/usr/local/bin` and the man page in
`/usr/local/share/man/man1`. `make uninstall` removes them.

### IRIX

`Makefile.irix` builds a static binary on IRIX, compiling the bundled libcdio
from source (the same approach as the Windows cross build).

```sh
make -f Makefile.irix
```

## About this fork

This fork lives at <https://github.com/sgi-demos/efsextract>. It adds
`Makefile.clang64`, a native Windows build path for the MSYS2 CLANG64
environment that links against the packaged libcdio rather than cross-compiling
or building libcdio from source.

```sh
git clone https://github.com/sgi-demos/efsextract.git
cd efsextract
```

## License

Copyright © 2023–2024 Jason Benaim. Licensed under the GNU GPL version 2 or
later. This is free software; you are free to change and redistribute it. There
is NO WARRANTY, to the extent permitted by law.

Upstream project: <https://jrra.zone/efsextract>

## See also

`iso-read(1)`, `isoinfo(1)`
