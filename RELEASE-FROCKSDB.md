## Build in Windows

Use Windows 64 bit machine (e.g. base AWS Windows instance: 4 cores, 16GB RAM, 40GB storage for build).

Install:
 * git
 * java 8
 * maven
 * Visual Studio Community 15 (2017)

Open git bash and run commands:

    git clone git@github.com:dataArtisans/frocksdb.git
    cd frocksdb
    git checkout FRocksDB-5.17.2 # release branch
    java/crossbuild/build-win.bat

The result native library is `build\java\Release\rocksdbjni-shared.dll`.
The result windows jar is `build\java\rocksdbjni_classes.jar`.

There is also a how-to in CMakeLists.txt.

## Build in PPC64LE

Use Ubuntu 16.04 (e.g. AWS instance 4 cores, 16GB RAM, 40GB storage for build).
Install git if not installed. If docker is installed, it might need to be removed.

Setup ppc64le docker machine ([source](https://developer.ibm.com/linuxonpower/2017/06/08/build-test-ppc64le-docker-images-intel/)):

    wget http://ftp.unicamp.br/pub/ppc64el/boot2docker/install.sh && chmod +x ./install.sh && ./install.sh -s
    docker-machine create -d qemu \
        --qemu-boot2docker-url=/home/ubuntu/.docker/machine/boot2docker.iso \
        --qemu-memory 8192 \
        --qemu-cache-mode none \
        --qemu-arch ppc64le \
        vm-ppc64le

Regenerate certs as suggested if it did not work at once.

Prepare docker machine to run rocksdbjni docker image for ppc64le build:

    eval $(docker-machine env vm-ppc64le)
    git clone git@github.com:dataArtisans/frocksdb.git
    cd frocksdb
    git checkout FRocksDB-5.17.2 # release branch
    docker-machine ssh vm-ppc64le mkdir -p `pwd`
    docker-machine scp -r . vm-ppc64le:`pwd`

Build frocksdb:

    make rocksdbjavastaticdockerppc64le
    docker-machine scp vm-ppc64le:`pwd`/java/target/librocksdbjni-linux-ppc64le.so java/target/.

The result native library is in `java/target/librocksdbjni-linux-ppc64le.so`.

## Final crossbuild in Mac OSX

Read how to Build cross jar for Mac OSX and linux as described in java/RELEASE.md but do not run it yet.

Run commands:

    make jclean clean
    mkdir -p java/target
    cp <path-to-windows-dll>/rocksdbjni-shared.dll java/target/librocksdbjni-win64.dll
    cp <path-to-windows-jar>/rocksdbjni_classes.jar java/target/rocksdbjni_classes.jar
    cp <path-to-ppc64le-lib-so>/librocksdbjni-linux-ppc64le.so java/target/librocksdbjni-linux-ppc64le.so
    FROCKSDB_VERSION=1.0 make frocksdbjavastaticrelease

## Push to maven central

