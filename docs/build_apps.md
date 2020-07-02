## Build applications without building the firmware

If you do not want to compile the complete firmware image, but still want to make changes to the basestation/client,
use this guide for how to cross-compile the applications.


### 1. Get ARM toolchain

!!! note "Already got Xilinx SDK installed?"
    If you already have the Xilinx SDK installed, you can skip this step.
    You may have to add the toolchain folder to your path, though.
    (add `export PATH=$PATH:/opt/Xilinx/SDK/2018.2/gnu/aarch32/lin/gcc-arm-linux-gnueabi/bin` 
    to `~/.profile`)

First, the arm toolchain has to be installed. Again, a more detailed instruction
can be found [here](https://wiki.analog.com/university/tools/pluto/devs/embedded_code).

We use linaro toolchain 7.2-2017.11, download it from [here](http://releases.linaro.org/components/toolchain/binaries/7.2-2017.11/arm-linux-gnueabihf/) (I've used x86_64 version).

Unpack it and copy it to /usr/local/bin/

```bash
tar -xf gcc-linaro-7.2.1-2017.11-x86_64_arm-linux-gnueabihf.tar.xz
sudo mv gcc-linaro-7.2.1-2017.11-x86_64_arm-linux-gnueabihf/ /usr/local/bin/gcc-linaro-7.2.1
```

Next, add the toolchain location to the path variable. To make this permanent, edit
` ~/.profile` and add the following lines:

```bash
# add linaro gcc to path
export PATH=$PATH:/usr/local/bin/gcc-linaro-7.2.1/bin
```
Log off and on to apply the changes.

### 2. Get custom Pluto sysroot

To successfully cross-compile, the pluto root directory has to be known. We use a custom sysroot, that
can be fetched in the release section of the [repository](https://github.com/HAMNET-Access-Protocol/HNAP4PlutoSDR/releases).

Download it and copy the folder to your home directory and set the environment variable **PLUTO_SYSROOT_DIR**.
It will be used by some of the following scripts.

```bash
export PLUTO_SYSROOT_DIR=$HOME/pluto-0.31.sysroot
```

### 3. Compile libraries

We have to compile libfftw on our own, because liquid-dsp requires
the single precision fftw3 lib, but adalm pluto only has the double precision version
installed by default.

Get the tarball from [http://fftw.org/download.html](http://fftw.org/download.html) and unpack it.
We are using FFTW 3.3.8.

```
wget http://fftw.org/fftw-3.3.8.tar.gz
tar xzfv fftw-3.3.8.tar.gz
cd fftw-3.3.8
./configure --host=arm-linux-gnueabihf --enable-float --enable-shared --prefix="$PLUTO_SYSROOT_DIR/usr/" CC="arm-linux-gnueabihf-gcc --sysroot=$PLUTO_SYSROOT_DIR/"
make
make install
```
This will install the shared library to the sysroot directory.

libfec and liquid-dsp can be configured and cross-compiled with the follwing commands

```bash
git clone --recursive https://github.com/HAMNET-Access-Protocol/HNAP4PlutoSDR.git

cd HNAP4PlutoSDR/libfec
./configure arm --host=arm-linux-gnueabihf  --prefix="$PLUTO_SYSROOT_DIR/usr/" CC="arm-linux-gnueabihf-gcc --sysroot=$PLUTO_SYSROOT_DIR/"
make
make install

cd ../liquid-dsp
apt install automake autoconf
./bootstrap.sh
./configure arm --host=arm-linux-gnueabihf  --prefix="$PLUTO_SYSROOT_DIR/usr/" CC="arm-linux-gnueabihf-gcc --sysroot=$PLUTO_SYSROOT_DIR/"
sed -i '/rpl_realloc/d' config.h
sed -i '/rpl_malloc/d' config.h
make
make install
```

NOTE: For liquid-dsp, the configure script does not correctly detect malloc and
realloc C functions, and thus make fails! We use the sed commands to fix this.


### 4. Compile apps

The apps are built using cmake. The target names are basestation client and client-calib.

```bash
mkdir cmake-build-release-arm
cd cmake-build-release-arm
cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_TOOLCHAIN_FILE=CmakeArmToolchain.cmake ..
make basestation
make client
make client-calib
```

### 5. Upload everything to pluto

Configure your Host PC as described in the **Host PC configuration** section.

To upload applications, do:
```
cd HNAP4PlutoSDR/cmake-build-release-arm/
scp basestation client client-calib plutosdr:/root/
```

Now, you can log in to the pluto and run the applications. They are located in /root/.

Note: every time Pluto reboots, The changed application files are lost. To make the changes
permanent, the firmware image has to be changed.

## Host PC configuration

In order to ease development with multiple plutos connected to your PC, you can configure your ssh client.

The host key of the pluto changes every time the device reboots, so for convenience
add the follwing to `~/.ssh/config`:

```
Host plutosdr
   HostName 192.168.2.1
   HostKeyAlias plutosdr
   StrictHostKeyChecking=no
   CheckHostIP no
   User root
   ChallengeResponseAuthentication no

```
Then the device can be accessed with `ssh plutosdr` (password: analog).

