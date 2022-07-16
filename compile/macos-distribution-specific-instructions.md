# MacOS Distribution Specific Instructions



## macOS Build Instructions and Notes

The commands in this guide should be executed in a Terminal application. The built-in one is located in

```
/Applications/Utilities/Terminal.app
```

### Preparation

Install the macOS command line tools:

```
xcode-select --install
```

When the popup appears, click `Install`.

Then install [Homebrew](https://brew.sh/).

### Dependencies

```
brew install automake libtool boost miniupnpc pkg-config python qt libevent qrencode fmt
```

If you run into issues, check [Homebrew's troubleshooting page](https://docs.brew.sh/Troubleshooting).

If you want to build the disk image with `make deploy` (.dmg / optional), you need RSVG:

```
brew install librsvg
```

The wallet support requires one or both of the dependencies ([_SQLite_](https://www.sqlite.org/) and [_Berkeley DB_](https://www.oracle.com/database/technologies/related/berkeleydb.html)) in the sections below.

**SQLite**

Usually, macOS installation already has a suitable SQLite installation. Also, the Homebrew package could be installed:

```
brew install sqlite
```

In that case the Homebrew package will prevail.

**Berkeley DB**

It is recommended to use Berkeley DB 4.8. If you have to build it yourself, you can use [this](https://github.com/litecoin-project/litecoin/blob/master/contrib/install\_db4.sh) script to install it like so:

```
./contrib/install_db4.sh .
```

from the root of the repository.

Also, the Homebrew package could be installed:

```
brew install berkeley-db4
```

### Build Tyzen Core

1.  Clone the Tyzen Core source code:

    ```
    git clone https://github.com/tyzen-tzn/tyzen
    cd tyzen
    ```
2.  Build Tyzen Core:

    Configure and build the headless Tyzen Core binaries as well as the GUI (if Qt is found).

    You can disable the GUI build by passing `--without-gui` to configure.

    ```
    ./autogen.sh
    ./configure
    make
    ```
3.  It is recommended to build and run the unit tests:

    ```
    make check
    ```
4.  You can also create a `.dmg` that contains the `.app` bundle (optional):

    ```
    make deploy
    ```

### Disable-wallet mode

When the intention is to run only a P2P node without a wallet, Tyzen Core may be compiled in disable-wallet mode with:

```
./configure --disable-wallet
```

Mining is also possible in disable-wallet mode using the `getblocktemplate` RPC call.

### Running

Tyzen Core is now available at `./src/`tyzend

Before running, you may create an empty configuration file:

```
mkdir -p "/Users/${USER}/Library/Application Support/Tyzen"

touch "/Users/${USER}/Library/Application Support/Tyzen/tyzen.conf"

chmod 600 "/Users/${USER}/Library/Application Support/Tyzen/tyzen.conf"
```

The first time you run tyzend, it will start downloading the blockchain. This process could take many hours, or even days on slower than average systems.

You can monitor the download process by looking at the debug.log file:

```
tail -f $HOME/Library/Application\ Support/Tyzen/debug.log
```

### Other commands:

```
./src/tyzend -daemon      # Starts the tyzen daemon.
./src/tyzen-cli --help    # Outputs a list of command-line options.
./src/tyzen-cli help      # Outputs a list of RPC commands when the daemon is running.
```

### Notes

* Tested on OS X 10.14 Mojave through macOS 11 Big Sur on 64-bit Intel processors only.
* Building with downloaded Qt binaries is not officially supported.
