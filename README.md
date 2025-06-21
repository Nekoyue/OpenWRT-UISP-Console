Credits:
- UISP Console 之创 - https://nyac.at/posts/uisp-console-chuang
- 给 UISP Console 适配 OpenWRT - https://nyac.at/posts/uisp-openwrt

# OpenWRT for OpenWRT console

## Build OpenWRT
Install build tools for your OS: https://openwrt.org/docs/guide-developer/toolchain/install-buildsystem

Clone this repo.

Then follow OpenWRT's build guide: https://openwrt.org/docs/guide-developer/toolchain/use-buildsystem#menuconfig
```shell
./scripts/feeds update -a
./scripts/feeds install -a

# Select:
# Target System - UISP Console
# Base system - swconfig
# LuCI
make menuconfig  

make -j$(nproc) defconfig download clean world
```

The image will be built at 
`./build_dir/target-aarch64_cortex-a57_musl/linux-uispconsole_/root.ext4`.

## Flash to SSD
TODO: driver for rtl8370mb is broken rn, i.e. port 1-8 is currently unavailable.

TTL/SSH into the device (default login: type `ubnt` + enter twice). Use scp to upload root.ext4 to the UISP console (e.g. /mnt/persistent). 
Then:

```shell
dd if=root.ext4 of=/dev/sda1 bs=4M
```

## Boot into OpenWRT
Connect to TTL. Press Esc when booting to enter uboot bootloader, run:

```shell
setenv rootargs root=/dev/sda1 rw rootwait
setenv bootargs $rootargs pci=pcie_bus_perf console=ttyS0,115200 panic=3 init=/sbin/init
ext4load usb 0 0x08000004 uImage.1
rtl83xx
bootm 0x08000004
```

To permanently boot from openwrt:
```shell
setenv rootargs root=/dev/sda1 rw rootwait
setenv bootargs $rootargs pci=pcie_bus_perf console=ttyS0,115200 panic=3 init=/sbin/init
setenv bootcmd 'rtl83xx; ext4load usb 0 0x08000004 uImage.1; bootm 0x08000004'
saveenv
```

## kexec (TODO)
Kernel module signing seems to be required in `4.19.152-al-linux-v10.2.0-v5.1.0`. Unable to load kexec module.
```
insmod: ERROR: could not insert module kexec_mod_arm64.ko: Required key not available
```

```shell
git clone https://github.com/fabianishere/kexec-mod.git
cd kexec-mod
scp ubnt@ubnt.lan:/proc/config.gz .
gunzip config.gz

git clone https://github.com/fabianishere/udm-kernel.git
cd udm-kernel
# v1.10.4
git checkout 89130462d8a36d535c1d0ce50a7a153ad1587865

cp ../config .config
export ARCH=arm64
make olddefconfig LOCALVERSION=
make modules_prepare LOCALVERSION=    

cd ../kernel
make KDIR=$(pwd)/udm-kernel
scp *.ko ubnt@ubnt.lan:/mnt/persistent

cd ../user
make
scp redir.so ubnt@ubnt.lan:/mnt/persistent
```

```shell
insmod kexec_mod_arm64.ko shim_hyp=1
insmod kexec_mod.ko
export LD_PRELOAD=./redir.so
kexec -l /boot/Image --reuse-cmdline
kexec -e
```


![OpenWrt logo](include/logo.png)

OpenWrt Project is a Linux operating system targeting embedded devices. Instead
of trying to create a single, static firmware, OpenWrt provides a fully
writable filesystem with package management. This frees you from the
application selection and configuration provided by the vendor and allows you
to customize the device through the use of packages to suit any application.
For developers, OpenWrt is the framework to build an application without having
to build a complete firmware around it; for users this means the ability for
full customization, to use the device in ways never envisioned.

Sunshine!

## Download

Built firmware images are available for many architectures and come with a
package selection to be used as WiFi home router. To quickly find a factory
image usable to migrate from a vendor stock firmware to OpenWrt, try the
*Firmware Selector*.

* [OpenWrt Firmware Selector](https://firmware-selector.openwrt.org/)

If your device is supported, please follow the **Info** link to see install
instructions or consult the support resources listed below.

## 

An advanced user may require additional or specific package. (Toolchain, SDK, ...) For everything else than simple firmware download, try the wiki download page:

* [OpenWrt Wiki Download](https://openwrt.org/downloads)

## Development

To build your own firmware you need a GNU/Linux, BSD or macOS system (case
sensitive filesystem required). Cygwin is unsupported because of the lack of a
case sensitive file system.

### Requirements

You need the following tools to compile OpenWrt, the package names vary between
distributions. A complete list with distribution specific packages is found in
the [Build System Setup](https://openwrt.org/docs/guide-developer/build-system/install-buildsystem)
documentation.

```
binutils bzip2 diff find flex gawk gcc-6+ getopt grep install libc-dev libz-dev
make4.1+ perl python3.7+ rsync subversion unzip which
```

### Quickstart

1. Run `./scripts/feeds update -a` to obtain all the latest package definitions
   defined in feeds.conf / feeds.conf.default

2. Run `./scripts/feeds install -a` to install symlinks for all obtained
   packages into package/feeds/

3. Run `make menuconfig` to select your preferred configuration for the
   toolchain, target system & firmware packages.

4. Run `make` to build your firmware. This will download all sources, build the
   cross-compile toolchain and then cross-compile the GNU/Linux kernel & all chosen
   applications for your target system.

### Related Repositories

The main repository uses multiple sub-repositories to manage packages of
different categories. All packages are installed via the OpenWrt package
manager called `opkg`. If you're looking to develop the web interface or port
packages to OpenWrt, please find the fitting repository below.

* [LuCI Web Interface](https://github.com/openwrt/luci): Modern and modular
  interface to control the device via a web browser.

* [OpenWrt Packages](https://github.com/openwrt/packages): Community repository
  of ported packages.

* [OpenWrt Routing](https://github.com/openwrt/routing): Packages specifically
  focused on (mesh) routing.

* [OpenWrt Video](https://github.com/openwrt/video): Packages specifically
  focused on display servers and clients (Xorg and Wayland).

## Support Information

For a list of supported devices see the [OpenWrt Hardware Database](https://openwrt.org/supported_devices)

### Documentation

* [Quick Start Guide](https://openwrt.org/docs/guide-quick-start/start)
* [User Guide](https://openwrt.org/docs/guide-user/start)
* [Developer Documentation](https://openwrt.org/docs/guide-developer/start)
* [Technical Reference](https://openwrt.org/docs/techref/start)

### Support Community

* [Forum](https://forum.openwrt.org): For usage, projects, discussions and hardware advise.
* [Support Chat](https://webchat.oftc.net/#openwrt): Channel `#openwrt` on **oftc.net**.

### Developer Community

* [Bug Reports](https://bugs.openwrt.org): Report bugs in OpenWrt
* [Dev Mailing List](https://lists.openwrt.org/mailman/listinfo/openwrt-devel): Send patches
* [Dev Chat](https://webchat.oftc.net/#openwrt-devel): Channel `#openwrt-devel` on **oftc.net**.

## License

OpenWrt is licensed under GPL-2.0
