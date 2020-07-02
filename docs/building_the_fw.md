## Building the HNAP Pluto firmware
This section will help you to create a custom firmware that includes all libraries, apps and autoconfiguration scripts for operating
the HNAP transceiver.
In order to guarantee a stable operation at high data-rates, the we use the linux realtime kernel patch. The following guide includes all steps to create a custom RT linux firmware with all necessary configurations for running as a client or as basestation.

### Prerequisites

#### Xilinx Vivado SDK

In order to be able to build the firmware, Xilinx Vivado Webpack 2018.2 has to be installed.
Get it from [here](https://www.xilinx.com/support/download/index.html/content/xilinx/en/downloadNav/vivado-design-tools.html).  
Version **2018.2** is listed in the Archive section. You have to create a Xilinx account to get the SDK, but the required webpack version is free.
During the software install, make sure the SDK and the Zynq 7000 software parts are installed (all other parts can be unselected).
You'll still need about 29 GB free disc space for this.

This guide assumes that you install Vivado and the SDK into `/opt/Xilinx/`.

#### Custom Pluto sysroot

To be able to compile the C applications, we need the pluto sysroot structure. The most recent
firmware and application require additional libraries. The pluto-sysroot folder that includes the
include files for these libs can be found under [releases](https://github.com/HAMNET-Access-Protocol/HNAP4PlutoSDR/releases): pluto-0.31-mod.sysroot.zip  
Unzip the directory to `~/pluto-0.31.sysroot`, then set the environment variable **PLUTO_SYSROOT_DIR**.
It will be used by some of the following scripts.

```bash
cd ~
wget https://github.com/HAMNET-Access-Protocol/HNAP4PlutoSDR/releases/download/v1.0.0/pluto-0.31-mod.sysroot.zip
unzip pluto-0.31-mod.sysroot.zip
export PLUTO_SYSROOT_DIR=$HOME/pluto-0.31.sysroot
```

### Download the Pluto firmware sources

Clone the Pluto firmware repository:
```
git clone --recurse-submodules -j8 https://github.com/analogdevicesinc/plutosdr-fw.git
```

### Apply the realtime patch for the linux kernel

Next, the linux realtime kernel patch will be applied.  
The current plutosdr-fw Version 0.31 uses Linux Kernel Version 4.19.0.
The corresponding patch is [patch-4.19-rt1.patch.gz](https://mirrors.edge.kernel.org/pub/linux/kernel/projects/rt/4.19/older/patch-4.19-rt1.patch.gz) from here: [https://kernel.org/pub/linux/kernel/projects/rt/4.19/older/](https://kernel.org/pub/linux/kernel/projects/rt/4.19/older/)

The patch must match the linux kernel version. If you did not download plutosdr-fw v0.31, check the linux kernel version used in plutosdr-fw as follows:
```
$ cd plutosdr-fw/
$ make -C linux ARCH=arm zynq_pluto_defconfig
$ make -C linux ARCH=arm menuconfig
```

A menuconfig window should appear with the linux kernel version in the title.

### Configure the linux kernel

Unpack the patch. Then apply it:
```
cd plutosdr-fw/linux/
patch -p1 < /path-to-patch/patch-4.19-rt1.patch
```

Now we will configure the linux kernel. This can be done with menuconfig.  
First, load the kernel config for the pluto device:

```
cd plutosdr-fw/
make -C linux ARCH=arm zynq_pluto_defconfig
```

Then edit it with menuconfig:
```
make -C linux ARCH=arm menuconfig
```

You will have do make the following changes:

- Go to: General Setup ---> Preemption Model ---> select Fully Preemptible Kernel (RT)
- Go to: Device Drivers ---> Network device support ---> Universal TUN/TAP device driver support. Type 'Y' to activate

If you cannot find the entries, do a search by typing '/' and go to the search result by typing the result number, e.g. '1'.

Go to 'Save' and use `arch/arm/configs/zynq_pluto_defconfig` to overwrite the default zynq_pluto configuration.

### Configure the userspace apps
Next, we will activate some useful userspace apps in buildroot. Load the pluto default configuration.

```
make -C buildroot ARCH=arm zynq_pluto_defconfig
make -C buildroot ARCH=arm menuconfig
```

Its the easiest to search for the apps with '/', select the result (e.g. '1') and activate it with 'y'.

- **tunctl**. Required. Must be installed to configgure the tap device
- **fftw-single**. Required. Search term 'single'. We need the single precision fftw library. Make sure to select 'BR2_PACKAGE_FFTW_SINGLE' and  not the legacy option.
- **libconfig**. Required. Search term 'libconfig'. A library to parse configuration files.
- **lldpd**. Required. Link Layer Discovery protocol. Used to broadcast callsigns.
- **iperf3**. Optional but recommended. Useful for bandwidth tests.
- **tcpdump**+**gdb** Optional. Useful for debugging.

For me, the External toolchain kernel header version was not correctly configured. If this is not correctly set, building the firmware will fail.
I had to set kernel header series 4.10.x, previously it was at 4.9.x. You can do this under 'Toolchain ---> External toolchain kernel headers series'.

In order to preinstall our own applications, we will create a rootfs overlay folder and tell buildroot to overlay this folder over the created filesystem.
Activate this with:

- `System configuration ---> Root filesystem overlay directories`

I choose `/home/lukas/plutosdr-fw/rootfs-overlay/`, keep in mind the chosen directory name when creating the overlay in the next steps.

Save the configuration by using the default path. Then overwrite the pluto target config with the following command:
```
make -C buildroot savedefconfig
```

Now we will fill the rootfs-overlay directory.
The git repository containing this readme comes with a script `config-firmware.sh`. It should do everything on its own.
Before you run it, edit it and check that the Variables `PLUTOSDR_FW_ROOTOVERLAY` and `PLUTO_SYSROOT_DIR` point to the correct paths.

Also, verify that `transceiver/CmakeArmToolchain.cmake` uses the correct sysroot path and the correct Xilinx toolchain path.

Then run:
```
./config-firmware.sh
```
The script will build all dependent libraries (libfec, liquid-dsp) and the C applications. Then it fills the rootoverlay with the applications, some startup scripts and FIR filter coefficients that we use.

### Build firmware image

Now everything is set up to build the firmware image:
```
cd plutosdr-fw/
make
```

That's it.

If you get an error like
> Incorrect selection of kernel headers: expected 4.14.x, got 4.10.x
  Incorrect selection of gcc version: expected 8.x, got 7.2.1

You have to clean your current buildroot configuration and have to start from scratch with the last saved config.
Go back to the buildroot userspace configuration and make sure to save the configuration at the end.

In the menuconfig step check your old settings from above and: 

- Go to: Toolchain -> External Toolchain kernel headers series 4.10.x
- Go to: Toolchain -> External Toolchain gcc version -> 7.x
```
cd plutosdr-fw/
make -C buildroot ARCH=arm clean
make -C buildroot ARCH=arm zynq_pluto_defconfig
make -C buildroot ARCH=arm menuconfig
make -C buildroot savedefconfig
make
```

After the build finished, the firmware image can be found in `plutosdr-fw/build/pluto.frm`.  
It can be installed using these [instructions](../installation).



