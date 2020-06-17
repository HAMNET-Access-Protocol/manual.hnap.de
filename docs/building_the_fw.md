## Creating the pluto firmware
This section will help you to create a custom firmware that includes all libraries, apps and autoconfiguration scripts for operating
the HNAP transceiver.
In order to guarantee a stable operation at high data-rates, the we use the linux realtime kernel patch. The following guide includes all steps to create a custom RT linux firmware with all necessary configurations for running as a client or as basestation.

### Prerequesites
In order to be able to build the firmware, Xilinx Vivado Webpack 2018.2 has to be installed.
Get it from here https://www.xilinx.com/support/download/index.html/content/xilinx/en/downloadNav/vivado-design-tools.html.
Version 2018.2 is listed in the Archive section. You have to create a Xilinx account to get the SDK, but the required webpack version is free.
During the software install, make sure the SDK and the Zynq 7000 software parts are installed (all other parts can be unselected).

This guide assumes that you install Vivado and the SDK into `/opt/Xilinx/`.

To be able to compile the C applications, we need the pluto sysroot structure. The most recent
firmware and application require additional libraries. The pluto-sysroot folder that includes the
include files for these libs can be found under [releases](https://github.com/HAMNET-Access-Protocol/HNAP4PlutoSDR/releases) in the repository: pluto-sysroot-0.31-mod.
copy the directory to `~/pluto-0.31.sysroot`.


### Download the firmware sources
Clone the pluto firmware repository
`git clone --recursive https://github.com/analogdevicesinc/plutosdr-fw.git`

Next, the linux realtime kernel patch will be applied. The current plutosdr-fw Version 0.31 uses Linux Kernel Version 4.14.0.
The corresponding patch is patch-4.14-rt1.patch.gz from here: https://kernel.org/pub/linux/kernel/projects/rt/4.14/older/

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
patch -p1 < /path-to-patch/patch-4.14-rt1.patch
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
- Go to: Kernel Features ---> Preemption Model ---> select Fully Preemptible Kernel (RT)
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

### Building the firmware

Now everything is set up to build the firmware image:
```
cd plutosdr-fw/
make
```

That's it.

If you get an error like
> Incorrect selection of kernel headers: expected 4.9.x, got 4.10.x

Go back to the buildroot userspace configuration and set the toolchain kernel headers to version 4.10.x. Make sure to save the configuration.

After the build finished, the firmware image can be found in `plutosdr-fw/build/pluto.frm`.



