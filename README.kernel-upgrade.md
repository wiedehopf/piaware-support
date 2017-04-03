# Upgrading to the raspberrypi.org kernels / firmware

PiAware sdcard images starting with 3.5.0 use the Pi Foundation kernels
(raspberrypi-kernel package). These are generally the most up to date
kernels available for the Pi.

PiAware sdcard images older than 3.5.0 use the raspbian.org kernels
(linux-image-*-rpi / linux-image-*-rpi2 packages). These kernels are older
and can be out of date and not support more recent Pi hardware.

If you are using a PiAware 3.0 / 3.1 / 3.3 sdcard image and you want to
change to using newer kernels, here's how to do it.

## WARNING

This process is quite fragile and if anything goes wrong you may be left
with an unbootable system. Make sure you back up anything you need in
advance, and avoid doing this on systems that you don't have easy physical
access to - you may need to reflash the sdcard to recover it.

A simpler approach if you don't mind reflashing the sdcard is to just reinstall
using the latest sdcard image, which is already set up to use newer kernels.

## Upgrade PiAware first

First, do a regular PiAware upgrade to 3.5.0 or later. Most importantly, you
need piaware-support 3.5.0 or newer.

## Do the upgrade

Everything here needs to be run as root (`sudo bash` for a root shell)

1. Remove the configuration that forces use of the raspbian.org packages:
```bash
rm -f /etc/apt/preferences.d/01vcgencmd.pref
```
2. Configure piaware-support to manage the kernel-specific parts of
`/boot/config.txt`:
```bash
echo "ENABLED=yes" >/etc/default/rpi-bootconfig
```
3. Install the new kernel and bootloader:
```bash
apt-get update
apt-get install raspberrypi-kernel raspberrypi-bootloader libraspberrypi-bin
```
4. Verify that there is an autogenerated section in config.txt that refers to
the newly installed kernel:
```bash
cat /boot/config.txt
```
At the end of the file, you should see a rpi-bootconfig autogenerated section
like this:

```
## start of rpi-bootconfig autogenerated section. Do not modify or remove this line.
[all]
[pi0]
kernel=kernel.img
initramfs initrd.img-4.4.50+ followkernel
[pi1]
kernel=kernel.img
initramfs initrd.img-4.4.50+ followkernel
[pi2]
kernel=kernel7.img
initramfs initrd.img-4.4.50-v7+ followkernel
[pi3]
kernel=kernel7.img
initramfs initrd.img-4.4.50-v7+ followkernel
[all]
## end of rpi-bootconfig autogenerated section. Do not modify or remove this line.
```
5. Remove the manual kernel section from the start of `/boot/config.txt`.
Edit /boot/config.txt with an editor of your choice:
```bash
nano /boot/config.txt
```
Delete the first part at the top that looks like this:
```
[pi0]
kernel=vmlinuz-4.4.0-1-rpi
initramfs initrd.img-4.4.0-1-rpi followkernel
[pi1]
kernel=vmlinuz-4.4.0-1-rpi
initramfs initrd.img-4.4.0-1-rpi followkernel
# to disable DeviceTree, uncomment the next line 
#device_tree=
[pi2]
kernel=vmlinuz-4.4.0-1-rpi2
initramfs initrd.img-4.4.0-1-rpi2 followkernel
[pi3]
kernel=vmlinuz-4.4.0-1-rpi2
initramfs initrd.img-4.4.0-1-rpi2 followkernel
```
6. Cross your fingers and reboot.
```bash
reboot
```
7. After the Pi has rebooted, check that you are using the new kernel:
```bash
uname -a
```
The reported version should have a trailing `+` or `-v7+` if you are on the new kernel:
```
Linux piaware 4.4.50-v7+ #970 SMP Mon Feb 20 19:18:29 GMT 2017 armv7l GNU/Linux
```
8. Optional: Remove the old kernel packages
```bash
apt-get remove 'linux-image-*'
```