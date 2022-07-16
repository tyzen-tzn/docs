# Linux Distribution Specific Instructions

## Ubuntu & Debian

**Dependency Build Instructions**

Build requirements:

```
sudo apt-get install build-essential libtool autotools-dev automake pkg-config bsdmainutils python3
```

Now, you can either build from self-compiled depends or install the required dependencies:

```
sudo apt-get install libevent-dev libboost-system-dev libboost-filesystem-dev libboost-test-dev libboost-thread-dev libfmt-dev
```

BerkeleyDB is required for the wallet.

Ubuntu and Debian have their own `libdb-dev` and `libdb++-dev` packages, but these will install BerkeleyDB 5.1 or later. This will break binary wallet compatibility with the distributed executables, which are based on BerkeleyDB 4.8. If you do not care about wallet compatibility, pass `--with-incompatible-bdb` to configure.

Otherwise, you can build from self-compiled `depends` (see above).

SQLite is required for the wallet:

```
sudo apt install libsqlite3-dev
```

Optional (see `--with-miniupnpc` and `--enable-upnp-default`):

```
sudo apt-get install libminiupnpc-dev
```

ZMQ dependencies (provides ZMQ API):

```
sudo apt-get install libzmq3-dev
```



GUI dependencies:

If you want to build tyzen-qt, make sure that the required packages for Qt development are installed. Qt 5 is necessary to build the GUI. To build without GUI pass `--without-gui`.

To build with Qt 5 you need the following:

```
sudo apt-get install libqt5gui5 libqt5core5a libqt5dbus5 qttools5-dev qttools5-dev-tools
```

libqrencode (optional) can be installed with:

```
sudo apt-get install libqrencode-dev
```

Once these are installed, they will be found by configure and a tyzen-qt executable will be built by default.

#### Fedora

**Dependency Build Instructions**

Build requirements:

```
sudo dnf install gcc-c++ libtool make autoconf automake libevent-devel boost-devel libdb4-devel libdb4-cxx-devel python3 fmt
```

Optional (see `--with-miniupnpc` and `--enable-upnp-default`):

```
sudo dnf install miniupnpc-devel
```

ZMQ dependencies (provides ZMQ API):

```
sudo dnf install zeromq-devel
```

To build with Qt 5 you need the following:

```
sudo dnf install qt5-qttools-devel qt5-qtbase-devel
```

libqrencode (optional) can be installed with:

```
sudo dnf install qrencode-devel
```

SQLite can be installed with:

```
sudo dnf install sqlite-devel
```

### Notes

The release is built with GCC and then "strip tyzend" to strip the debug symbols, which reduces the executable size by about 90%.

### miniupnpc

[miniupnpc](https://miniupnp.tuxfamily.org/) may be used for UPnP port mapping. It can be downloaded from [here](https://miniupnp.tuxfamily.org/files/). UPnP support is compiled in and turned off by default. See the configure options for upnp behavior desired:

```
--without-miniupnpc      No UPnP support miniupnp not required
--disable-upnp-default   (the default) UPnP support turned off by default at runtime
--enable-upnp-default    UPnP support turned on by default at runtime
```

### Setup and Build Example: Arch Linux



This example lists the steps necessary to setup and build a command line only, non-wallet distribution of the latest changes on Arch Linux:

```
pacman -S git base-devel boost libevent python
git clone https://github.com/tyzen-tzn/tyzen.git
cd tyzen/
./autogen.sh
./configure --disable-wallet --without-gui --without-miniupnpc
make check
```

Note: Enabling wallet support requires either compiling against a Berkeley DB newer than 4.8 (package `db`) using `--with-incompatible-bdb`, or building and depending on a local version of Berkeley DB 4.8. The readily available Arch Linux packages are currently built using `--with-incompatible-bdb` according to the [PKGBUILD](https://projects.archlinux.org/svntogit/community.git/tree/bitcoin/trunk/PKGBUILD). As mentioned above, when maintaining portability of the wallet between the standard Tyzen Core distributions and independently built node software is desired, Berkeley DB 4.8 must be used.

### ARM Cross-compilation

These steps can be performed on, for example, an Ubuntu VM. The depends system will also work on other Linux distributions, however the commands for installing the toolchain will be different.

Make sure you install the build requirements mentioned above. Then, install the toolchain and curl:

```
sudo apt-get install g++-arm-linux-gnueabihf curl
```

To build executables for ARM:

```
cd depends
make HOST=arm-linux-gnueabihf NO_QT=1
cd ..
./autogen.sh
./configure --prefix=$PWD/depends/arm-linux-gnueabihf --enable-glibc-back-compat --enable-reduce-exports LDFLAGS=-static-libstdc++
make
```
