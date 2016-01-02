# Compile Monero 0.9 on Ubuntu 15.10 x86_64
[Monero 0.9](https://github.com/monero-project/bitmonero/releases/tag/v0.9.0) finally arrived!

This is good chance to update this example for Monero 0.9. Specifically, the example shows how to
compile current github version of [Monero](https://getmonero.org/), as of 03 Jan 2015,
on Ubuntu 15.10 x86_64.

## Preparation
Before proceeding to the compilation, the following packages are required:

 ```bash
# update ubuntu's repository
sudo apt-get update

#install git
sudo apt-get install git

# install dependencies
sudo apt-get install build-essential cmake libboost-all-dev miniupnpc libunbound-dev graphviz doxygen libdb5.3++-dev
 ```

## Compilation
Having the dependencies, we can download the current Monero version and compile it as follows:

```bash
# download the latest bitmonero source code from github
git clone https://github.com/monero-project/bitmonero.git

# go into bitmonero folder
cd bitmonero/

# compile
make # or make -j number_of_threads, e.g., make -j 2
```
## Installation (optional)
After successful compilation, the Monero binaries should be located in `./build/release/bin` as shown
below:

```bash
./bitmonero
├── build
│   └── release
│       ├── bin
│       │   ├── bitmonerod
│       │   ├── blockchain_converter
│       │   ├── blockchain_dump
│       │   ├── blockchain_export
│       │   ├── blockchain_import
│       │   ├── cn_deserialize
│       │   ├── connectivity_tool
│       │   ├── simpleminer
│       │   └── simplewallet

# other subfolders than bin not shown here.
```

I usually move the binaries into `/opt/bitmonero/` folder. This can be done
as follows:

```bash
# optional
sudo mkdir /opt/bitmonero
sudo mv ./build/release/bin/* /opt/bitmonero/
```

This should result in:
```bash
/opt/bitmonero
├── bitmonerod
├── blockchain_converter
├── blockchain_dump
├── blockchain_export
├── blockchain_import
├── cn_deserialize
├── connectivity_tool
├── showmixins
├── simpleminer
└── simplewallet
```

Now we can start the Monero daemon, i.e. `bitmonerod`, and let it
download the blockchain and synchronize itself with the Monero network. After that, you can run your the `simplewallet`.

```bash
# launch the Monero daemon and let it synchronize with the Monero network
/opt/bitmonero/bitmonerod

# launch the Monero wallet
/opt/bitmonero/simplewallet
```

## Monero C++11 development (optional)
If you want to develop your own C++11 programs on top of Monero 0.9,
Monero's static libraries and headers will be needed. Below is shown
how they can be setup for use to write your own C++11 programs based
on Monero. An example of such a program is  [access-blockchain-in-cpp](https://github.com/moneroexamples/access-blockchain-in-cpp).


### Monero static libraries

When the compilation finishes, a number of static Monero libraries
should be generated. We will need them to link against.

Since they are spread out over different subfolders of the `./build/` folder, it is easier to just copy them into one folder. I assume that
 `/opt/bitmonero-dev/libs` is the folder where they are going to be copied to.

```bash
# create the folder
sudo mkdir -p /opt/bitmonero-dev/libs

# find the static libraries files (i.e., those with extension of *.a)
# and copy them to /opt/bitmonero-dev/libs
# assuming you are still in bitmonero/ folder which you downloaded from
# github
sudo find ./build/ -name '*.a' -exec cp {} /opt/bitmonero-dev/libs  \;
 ```

 This should results in the following file structure:

 ```bash
 /opt/bitmonero-dev/
└── libs
    ├── libblockchain_db.a
    ├── libblocks.a
    ├── libcommon.a
    ├── libcrypto.a
    ├── libcryptonote_core.a
    ├── libcryptonote_protocol.a
    ├── libdaemonizer.a
    ├── libgtest.a
    ├── libgtest_main.a
    ├── liblmdb.a
    ├── libminiupnpc.a
    ├── libmnemonics.a
    ├── libotshell_utils.a
    ├── libp2p.a
    ├── librpc.a
    └── libwallet.a
```

### Monero headers

Now we need to get Monero headers, as this is our interface to the
Monero libraries. Folder `/opt/bitmonero-dev/headers` is assumed
to hold the headers.

```bash
# create the folder
sudo mkdir -p /opt/bitmonero-dev/headers

# find the header files (i.e., those with extension of *.h)
# and copy them to /opt/bitmonero-dev/headers.
# but this time the structure of directories is important
# so rsync is used to find and copy the headers files
sudo rsync -zarv --include="*/" --include="*.h" --exclude="*" --prune-empty-dirs ./ /opt/bitmonero-dev/headers
 ```

This should results in the following file structure:

```bash
# only src/ folder with up to 3 level nesting is shown

/opt/bitmonero-dev/headers/src/
├── blockchain_db
│   ├── berkeleydb
│   │   └── db_bdb.h
│   ├── blockchain_db.h
│   ├── db_types.h
│   └── lmdb
│       └── db_lmdb.h
├── blockchain_utilities
│   ├── blockchain_utilities.h
│   ├── blocksdat_file.h
│   ├── bootstrap_file.h
│   ├── bootstrap_serialization.h
│   └── fake_core.h
├── blocks
│   └── blocks.h
├── common
│   ├── base58.h
│   ├── boost_serialization_helper.h
│   ├── command_line.h
│   ├── dns_utils.h
│   ├── http_connection.h
│   ├── i18n.h
│   ├── int-util.h
│   ├── pod-class.h
│   ├── rpc_client.h
│   ├── scoped_message_writer.h
│   ├── unordered_containers_boost_serialization.h
│   ├── util.h
│   └── varint.h
├── crypto
│   ├── blake256.h
│   ├── chacha8.h
│   ├── crypto.h
│   ├── crypto_ops_builder
│   │   ├── api.h
│   │   ├── crypto_int32.h
│   │   ├── crypto-ops.h
│   │   ├── crypto_sign.h
│   │   ├── crypto_uint32.h
│   │   ├── crypto_verify_32.h
│   │   ├── include
│   │   ├── ref10
│   │   ├── ref10CommentedCombined
│   │   └── sha512.h
│   ├── crypto-ops.h
│   ├── generic-ops.h
│   ├── groestl.h
│   ├── groestl_tables.h
│   ├── hash.h
│   ├── hash-ops.h
│   ├── initializer.h
│   ├── jh.h
│   ├── keccak.h
│   ├── oaes_config.h
│   ├── oaes_lib.h
│   ├── random.h
│   ├── skein.h
│   └── skein_port.h
├── cryptonote_config.h
├── cryptonote_core
│   ├── account_boost_serialization.h
│   ├── account.h
│   ├── blockchain.h
│   ├── blockchain_storage_boost_serialization.h
│   ├── blockchain_storage.h
│   ├── checkpoints_create.h
│   ├── checkpoints.h
│   ├── connection_context.h
│   ├── cryptonote_basic.h
│   ├── cryptonote_basic_impl.h
│   ├── cryptonote_boost_serialization.h
│   ├── cryptonote_core.h
│   ├── cryptonote_format_utils.h
│   ├── cryptonote_stat_info.h
│   ├── difficulty.h
│   ├── hardfork.h
│   ├── miner.h
│   ├── tx_extra.h
│   ├── tx_pool.h
│   └── verification_context.h
├── cryptonote_protocol
│   ├── blobdatatype.h
│   ├── cryptonote_protocol_defs.h
│   ├── cryptonote_protocol_handler_common.h
│   └── cryptonote_protocol_handler.h
├── daemon
│   ├── command_line_args.h
│   ├── command_parser_executor.h
│   ├── command_server.h
│   ├── core.h
│   ├── daemon.h
│   ├── executor.h
│   ├── p2p.h
│   ├── protocol.h
│   ├── rpc_command_executor.h
│   └── rpc.h
├── daemonizer
│   ├── daemonizer.h
│   ├── posix_fork.h
│   ├── windows_service.h
│   └── windows_service_runner.h
├── miner
│   ├── simpleminer.h
│   ├── simpleminer_protocol_defs.h
│   └── target_helper.h
├── mnemonics
│   ├── electrum-words.h
│   ├── english.h
│   ├── german.h
│   ├── italian.h
│   ├── japanese.h
│   ├── language_base.h
│   ├── old_english.h
│   ├── portuguese.h
│   ├── russian.h
│   ├── singleton.h
│   └── spanish.h
├── p2p
│   ├── net_node_common.h
│   ├── net_node.h
│   ├── net_peerlist_boost_serialization.h
│   ├── net_peerlist.h
│   ├── p2p_protocol_defs.h
│   └── stdafx.h
├── platform
│   ├── mingw
│   │   └── alloca.h
│   └── msc
│       ├── alloca.h
│       ├── inline_c.h
│       ├── stdbool.h
│       └── sys
├── rpc
│   ├── core_rpc_server_commands_defs.h
│   ├── core_rpc_server_error_codes.h
│   └── core_rpc_server.h
├── serialization
│   ├── binary_archive.h
│   ├── binary_utils.h
│   ├── crypto.h
│   ├── debug_archive.h
│   ├── json_archive.h
│   ├── json_utils.h
│   ├── serialization.h
│   ├── string.h
│   ├── variant.h
│   └── vector.h
├── simplewallet
│   ├── password_container.h
│   └── simplewallet.h
└── wallet
    ├── wallet2.h
    ├── wallet_errors.h
    ├── wallet_rpc_server_commands_defs.h
    ├── wallet_rpc_server_error_codes.h
    └── wallet_rpc_server.h
```

Full `/opt/bitmonero-dev/` tree is [here](https://github.com/moneroexamples/compile-monero-09-ubuntu-1510/blob/master/res/full_tree_bitmonero-dev.txt).

## How can you help?

Constructive criticism, code and website edits are always good. They can be made through github.

Some Monero are also welcome:
```
48daf1rG3hE1Txapcsxh6WXNe9MLNKtu7W7tKTivtSoVLHErYzvdcpea2nSTgGkz66RFP4GKVAsTV14v6G3oddBTHfxP6tU
```
