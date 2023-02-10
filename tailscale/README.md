# Running Tailscale on the DevTerm-R01

These are my notes for running [Tailscale](https://tailscale.com/) for the DevTerm-R01.

Note this is pre Linux 6.2, which will have proper [Allwinner D1](https://www.phoronix.com/news/Linux-6.2-Sun4i-A100-D1) support rendering many of these instructions obsolete.

Tailscale will not install due to missing the `tun.ko` module which is not included in the default R-01 kernel. Trying to start `tailscaled` will give the following error in `/var/log/syslog`

```
Jan 26 02:25:17 localhost systemd[1]: Started Tailscale node agent.
Jan 26 02:25:17 localhost tailscaled[1074605]: wgengine.NewUserspaceEngine(tun "tailscale0") ...
Jan 26 02:25:17 localhost tailscaled[1074605]: Linux kernel version: 5.4.61
Jan 26 02:25:17 localhost tailscaled[1074605]: is CONFIG_TUN enabled in your kernel? `modprobe tun` failed with: modprobe: FATAL: Module tun not found in directory /lib/modules/5.4.61
Jan 26 02:25:19 localhost tailscaled[1074605]: tun module not loaded nor found on disk
Jan 26 02:25:19 localhost tailscaled[1074605]: wgengine.NewUserspaceEngine(tun "tailscale0") error: tstun.New("tailscale0"): CreateTUN("tailscale0") failed; /dev/net/tun does not exist
Jan 26 02:25:19 localhost tailscaled[1074605]: flushing log.
Jan 26 02:25:19 localhost tailscaled[1074605]: logger closing down
```


## Building Kernel

It is faster to cross-compile kernel on x86 in a Ubuntu 22.04 VM, if done on the R-01 it could take a very long time.

Also see: https://github.com/clockworkpi/DevTerm/wiki/Create-DevTerm-R01-OS-image-from-scratch#how-to-compile-kernel

### Install Required Packages

```
sudo apt install gcc-11-riscv64-linux-gnu binutils-riscv64-linux-gnu qemu-user-static build-essential git wget curl vim libncurses-dev flex automake autoconf bison libssl-dev
```

### Building the kernel

1. Clone git source from ccu

```
git clone https://github.com/cuu/last_linux-5.4.git
```

2. Setup Crossbuild toolchain in `/opt`
https://github.com/cuu/toolchain-thead-glibc

```
tar -xvzf riscv64-glibc-gcc-thead_20200702.tar.gz
```

3. Edit `~/.baschr` to include path

```
export PATH=/home/micheal/riscv64-glibc-gcc-thead_20200702/bin/:$PATH
```

4. Copy `config.gz` for running kernel config from devterm

```
scp cpi@192.168.7.245:/proc/config.gz
mv config.gz git/last_linux-5.4/
gunzip config.gz
mv config .config
```

4. Make menuconfig to select `CONFIG_TUN` driver

```
make CROSS_COMPILE=riscv64-unknown-linux-gnu- ARCH=riscv menuconfig
```

5. This will load in `.config` copied from the devterm and populate everything

6. Select TUN/TAP device support

see: https://tailscale.com/

```
Linux Kernel Configuration
└─> Device Drivers
└─> Network device support
└─> Universal TUN/TAP device driver support
```

7. Compile kernel

Edit `./m.sh` to include `LOCALVERSION=` after each `make` in order to avoid adding a `+` to the kernel version, which can cause `version magic` errors.

```
mkdir -p test/boot
./m.sh
```

### Adding Tunnel Drivers to R-01

Kernel modules are in `/home/micheal/git/last_linux-5.4/test/rootfs/lib/modules/5.4.61`

Create tarball with these to copy to the DevTerm

```
cd /home/micheal/git/last_linux-5.4/test/rootfs/lib/modules
tar -cvzf 5.4.61.modules.tar.gz 5.4.61/
```

On DevTerm, backup original modules

```
cd /lib/modules
sudo mv 5.4.61 5.4.61.orig
```

Untar new modules and rename directory

```
cd /lib/modules
sudo tar -xvzf ~/5.4.61.modules.tar.gz
```

Load module

```
modprobe tun
```

This will show the module loaded properly in `dmesg`

```
tun: Universal TUN/TAP device driver, 1.6
```

Start `tailscaled`

```
sudo service tailscaled start
```

Configure tailscale

```
sudo tailscale up
```
