# SHIELD: Encrypting Persistent Data of LSM-KVS from Monolithic to Disaggregated Datacenters

## Setup

### Pre-requisites

1. OpenSSL
2. Java 11
3. Maven
4. Node.js
5. RocksDB required libraries (refer [INSTALL.md](./INSTALL.md))
6. grpc (Disaggregated setups only)
7. HDFS (Disaggregated setups only)

## Install - Monolithic

### SHIELD
1. The code for SHIELD is integrated into RocksDB. 

2. Before building RocksDB, we need to build iotauth. We have forked iotauth from its official repository and made modifications to the codebase to work best with our deployment:
    ```bash
    pushd ./iotauth/examples/
    ./cleanAll.sh
    ./generateAll.sh -g ./configs/rocksdb.graph
    cd ../auth/auth-server
    java -jar target/auth-server-jar-with-dependencies.jar -p ../properties/exampleAuth101.properties
    popd
    ```

3. Change the configurations

    3.1 Search for "/path/to/db/" and replace with location to save the data encryption keys. 

    3.2 Go to ./examples/c_client.config and update the paths for the public and private keys (they are located in iotauth/entity/auth_certs (and) iotauth/entity/credentials/certs/net1)

    3.3 Go to options.cc file (./options/options.cc), search for "/path/to/c_client.config" and update the path to point to the c_clients/config above. 

4. Build using CMake:
    ```bash
    mkdir build
    cd build
    cmake -DROCKSDB_BUILD_SHARED=OFF -DWITH_TESTS=OFF -DWITH_HDFS=OFF -DWITH_EXAMPLES=OFF -DWITH_SSTLIB=ON -DWITH_CSA=OFF -DCMAKE_BUILD_TYPE=Release ..
    make -j64
    ```

### PineFS
1. The code for PineFS is located in /PineFS. This example will build both SHIELD and PineFS at the same time, please make sure you have setup completed the steps for SHIELD above. 

2. Move the folder to the plugins directory: `mv pinefs/ ./plugin/`

3. Build using CMake:
    ```bash
    mkdir build
    cd build
    cmake -DROCKSDB_BUILD_SHARED=OFF -DWITH_TESTS=OFF -DWITH_HDFS=OFF -DWITH_EXAMPLES=OFF -DWITH_SSTLIB=ON -DWITH_CSA=OFF -DCMAKE_BUILD_TYPE=Release -DROCKSDB_PLUGINS=pinefs ..
    make -j64
    ```

### Usage in Monolithic

```bash
#Unencrypted RocksDB
./db_bench --benchmarks=fillrandom --compression_type=none

#SHIELD
./db_bench --benchmarks=fillrandom --compression_type=encrypt --wal_compression=encrypt

#PineFS
./db_bench --benchmarks=fillrandom --fs_uri="provider={id=AES;hex_instance_key=0123456789ABCDEF0123456789ABCDEF0123456789ABCDEF0123456789ABCDEF;method=AES256CTR};id=EncryptedFileSystem" --compression_type=none
```

## Install - Disaggregated Storage

Please make sure you have a second server setup and have HDFS deployed on that server. Make sure you install the [RocksDB HDFS plugin](https://github.com/riversand963/rocksdb-hdfs-env).

1. Build using CMake:
    ```bash
    mkdir build
    cd build
    cmake -DROCKSDB_BUILD_SHARED=OFF -DWITH_TESTS=OFF -DWITH_HDFS=ON -DWITH_EXAMPLES=OFF -DWITH_SSTLIB=ON -DWITH_CSA=OFF -DCMAKE_BUILD_TYPE=Release -DROCKSDB_PLUGINS=hdfs ..
    make -j64
    ```

### Usage

```bash
#Unencrypted RocksDB
./db_bench --benchmarks=fillrandom --compression_type=none --env_uri="path/to/hdfs"

#SHIELD
./db_bench --benchmarks=fillrandom --compression_type=encrypt --wal_compression=encrypt --env_uri="path/to/hdfs"
```

## Install - Offloaded Compaction

Please follow the  the following commands on both servers you have.

1. Update the hdfs (hdfs_address) and the offloaded compaction path (csa_address) in the options.h file (options/options.h)

2. Build using CMake:
    ```bash
    mkdir build
    cd build
    cmake -DROCKSDB_BUILD_SHARED=OFF -DWITH_TESTS=OFF -DWITH_HDFS=ON -DWITH_EXAMPLES=OFF -DWITH_SSTLIB=ON -DWITH_CSA=ON -DCMAKE_BUILD_TYPE=Release -DROCKSDB_PLUGINS=pinefs ..
    make -j64
    ```

> On Server 2 you need to update your c_client.config file to have the correct IP address of the server where you have deployed iotauth. 

### Usage

> On server 2, run the command ./csa_server 

On server 1, run the following, commands:

```bash
#Unencrypted RocksDB
./db_bench --benchmarks=fillrandom --compression_type=none --allow_remote_compaction

#SHIELD
./db_bench --benchmarks=fillrandom --compression_type=encrypt --wal_compression=encrypt --allow_remote_compaction
```