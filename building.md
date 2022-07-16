---
description: >-
  The following are developer notes on how to build Tyzen Core on your native
  platform. They are not complete guides, but include notes on the necessary
  libraries, compile flags, etc.
---

# Building

## UNIX BUILD

Some notes on how to build Tyzen Core in Unix.

(For BSD specific instructions, see `build-*bsd.md` in this directory.)

#### NOTE

Always use absolute paths to configure and compile Tyzen Core and the dependencies. For example, when specifying the path of the dependency:

```
../dist/configure --enable-cxx --disable-shared --with-pic --prefix=$BDB_PREFIX
```

Here BDB\_PREFIX must be an absolute path - it is defined using $(pwd) which ensures the usage of the absolute path.

### To Build

```
./autogen.sh
./configure
make
make install # optional
```

This will build tyzen-qt as well, if the dependencies are met.

### Dependencies

These dependencies are required:

| Library  | Purpose    | Description                                 |
| -------- | ---------- | ------------------------------------------- |
| libboost | Utility    | Library for threading, data structures, etc |
| libevent | Networking | OS independent asynchronous networking      |

Optional dependencies:

| Library     | Purpose          | Description                                                                                                |
| ----------- | ---------------- | ---------------------------------------------------------------------------------------------------------- |
| miniupnpc   | UPnP Support     | Firewall-jumping support                                                                                   |
| libdb4.8    | Berkeley DB      | Wallet storage (only needed when wallet enabled)                                                           |
| qt          | GUI              | GUI toolkit (only needed when GUI enabled)                                                                 |
| libqrencode | QR codes in GUI  | Optional for generating QR codes (only needed when GUI enabled)                                            |
| univalue    | Utility          | JSON parsing and encoding (bundled version will be used unless --with-system-univalue passed to configure) |
| libzmq3     | ZMQ notification | Optional, allows generating ZMQ notifications (requires ZMQ version >= 4.0.0)                              |
| sqlite3     | SQLite DB        | Optional, wallet storage (only needed when wallet enabled)                                                 |

### Memory Requirements

C++ compilers are memory-hungry. It is recommended to have at least 1.5 GB of memory available when compiling Tyzen Core. On systems with less, gcc can be tuned to conserve memory with additional CXXFLAGS:

```
./configure CXXFLAGS="--param ggc-min-expand=1 --param ggc-min-heapsize=32768"
```

Alternatively, or in addition, debugging information can be skipped for compilation. The default compile flags are `-g -O2`, and can be changed with:

```
./configure CXXFLAGS="-O2"
```

Finally, clang (often less resource hungry) can be used instead of gcc, which is used by default:

```
./configure CXX=clang++ CC=clang
```

