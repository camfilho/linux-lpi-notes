# Determine and configure hardware settings

BIOS (Basic Input/Output System, the standard for firmware containing the basic configuration routines found in the x86 motherboards. From the end of the first decade of the 2000s, machines based on the x86 architecture started to replace the BIOS with a new implementation called UEFI (Unified Extensible Firmware Interface)

> End of first decade of the 2000s, x86 machines started to replace the BIOS with UEFI

## Device activation

In the BIOS setup utility it is possible to enable and disable integrated periphals.

If the machine is equipped with many storage devices, it is important to define which one has the correct bootloader and should be the first entry in the device boot order.

## Device inspection in Linux
There are two basic ways to identify hardware resources on a Linux system: to use specialized commands or to read specific file inside special filesystems.

### Commands for Inspection
The two essential commands to identify connected devices in a Linux system are:

**lspci** - Show all devices connected. PCI(Peripheral Component Interconnect bus)

**lsusb** - Lists USB (Universal Serial Bus) devices

`lspci` and `lsusb` list of all PCi and USB devices identified by the operating system.
However, it might not be fully operation yet becayse every hardware part requires a software component to control the corresponding device. `kernel modules`

>  kernel module related to hardware devices are called `drivers`


```bash
$ **lspci**
01:00.0 VGA compatible controller: NVIDIA Corporation GM107 [GeForce GTX 750 Ti] (rev a2)
04:02.0 Network controller: Ralink corp. RT2561/RT61 802.11g PCI
04:04.0 Multimedia audio controller: VIA Technologies Inc. ICE1712 [Envy24] PCI Multi-Channel I/O Controller (rev 02)
04:0b.0 FireWire (IEEE 1394): LSI Corporation FW322/323 [TrueFire] 1394a Controller (rev 70)
```

* The hexadecimal numbers at the beginning of each line are the unique addresses of the corresponding PCI device.


The command `lspci` shows more details about a specific device if its address is given with option `-s`, accompanied by the option `-v`:
```bash
**lspci -s 04:02.0 -v**
04:02.0 Network controller: Ralink corp. RT2561/RT61 802.11g PCI
    Subsystem: Linksys WMP54G v4.1
    Flags: bus master, slow devsel, latency 32, IRQ 21
    Memory at e3100000 (32-bit, non-prefetchable) [size=32K]
    Capabilities: [40] Power Management version 2
    kernel driver in use: **rt61pci**
```

The kernel module can be identified with the **lspci -s {{address of the PCI device}} -v** by looking at the __kernel driver in use__


Another way to identify which kernel drive in use:
```bash
$ **lspci -s 01:00.0 -k**
01:00.0 VGA compatible controller: NVIDIA Corporation GM107 [GeForce GTX 750 Ti] (rev a2)
    kernel driver in use: nvidia
    kernel modules: nouveau, nvidia_drm, nvidia
```


Command `lsusb` is similar to `lspci`, but lists USB information exclusively.

```bash
$ **lsusb**
Bus 001 Device 029: ID 1781:0c9f Multiple Vendors USBtiny
Bus 001 Device 028: ID 093a:2521 Pixart Imaging, Inc. Optical Mouse
Bus 001 Device 020: ID 1131:1001 Integrated System Solution Corp. KY-BT100 Bluetooth Adapter
Bus 001 Device 011: ID 04f2:0402 Chicony Electronics Co., Ltd Genius LuxeMate i200 Keyboard
Bus 001 Device 007: ID 0424:7800 Standard Microsystems Corp.
Bus 001 Device 003: ID 0424:2514 Standard Microsystems Corp. USB 2.0 Hub
Bus 001 Device 002: ID 0424:2514 Standard Microsystems Corp. USB 2.0 Hub
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
```

A specific device can be selected for inspection by providing its ID to the option -d

`$ lsusb -v -d {{device ID}}`

`lsmod` shows all currently loaded modules

```bash
$ **lsmod**
Module                  Size  Used by
kvm_intel             138528  0
kvm                   421021  1 kvm_intel
iTCO_wdt               13480  0
```


The output of `lsmod` is divided into three columns

Module - name
Size - size occupied in the RAM by the module
used by - depending modules

Some modules require other modules to work properly, as is the case with modules for audio devices:

```
$ **lsmod | fgrep -i snd_hda_intel**
snd_hda_intel          42658  5
snd_hda_codec         155748  3 snd_hda_codec_hdmi,snd_hda_codec_via,snd_hda_intel
snd_pcm                81999  3 snd_hda_codec_hdmi,snd_hda_codec,snd_hda_intel
snd_page_alloc         13852  2 snd_pcm,snd_hda_intel
snd                    59132  19 snd_hwdep,snd_timer,snd_hda_codec_hdmi,snd_hda_codec_via,snd_pcm,snd_seq,snd_hda_codec,snd_hda_intel,snd_seq_device

```

The third column, `Used by`, shows the modules that require the module in the first column to properly work.
Many modules of the Linux sound architecture, prefixed by `snd`, are interdependent. 

> When looking for problems during system diagnostics, it may be useful to unload specific modules currently loaded.


Command `modprobe` can be used to both load and to unload kernel modules as long as it's not being used by a running process.

``` # modprobe -r snd-hda-intel```

It is possible to change module parameters. Each module accepts specific parameters.


`**/etc/modprobe.conf**` - persist parameters by including them
`**/etc/modprobe.d/{{module name}}.con**` - persist parameters by including them

`**modinfo**` - display all available parameters

### Block loading module

add module to `**/etc/modprobe.d/blacklist.conf`


### Information Files and Device files

- The commands lspci, lsusb and lsmod act as front-ends to read hardware information store by the operating system.
- The `/proc` directory contains files with information regarding running processes and hardware resources

1. `/proc/cpuinfo` - List information about CPUs found by the OS
2. /proc/interrupts - List interrupts per IO device for each CPU
3. `/proc/ioports` - Lists currently registered Input/Output port regions in use.
4. `/proc/dma` - Direct Memory Access - list DMA channels in use.

### /sys vs /proc vs /dev

The /sys directory has the specific purpose of storing device information and kernel data related to hardware, whilst /proc also contains information about various kernel data structures, including running processes and configuration.

Every file inside /dev is associated with a ssystem device. A legacy IDE hard drive, for example, when connected to the motherboard's first IDE channel is represented by the file `/dev/hda` Every partition in this disk will be identified by the `/dev/hda1`, `/dev/hda2` up to the last partition found.

Removable devices are handled by the `udev` subsystem

### storage device

Storage devices are referred as **block devices**

From Linux kernel version 2.4 onwards, most storage devices are now identified as if they were SCSI devices, regardless of their hardware type. IDE, SSD and USB block devices will be prefixed by `sd`. For IDE disks, the `sd` prefix will be used, but the third letter will be chosen depending on whether the drive is a master or slave (in the first IDE channel, master will be `sda` and slave will be `sdb`). Partitions are listed numerically. Paths `/dev/sda1`, `/dev/sda2`, etc. are used for the first and second partitions of the block device identified first and `/dev/sdb1`, `/dev/sdb2`, etc. used to identify the first and second partitions of the block device identified second. The exception to this pattern occurs with memory cards (SD cards) and NVMe devices (SSD connected to the PCI Express bus). For SD cards, the paths `/dev/mmcblk0p1`, `/dev/mmcblk0p2`, etc. are used for the first and second partitions of the device identified first and `/dev/mmcblk1p1`, `/dev/mmcblk1p2`, etc. used to identify the first and second partitions of the device identified second. NVMe devices receive the prefix `nvme`, as in `/dev/nvme0n1p1` and `/dev/nvme0n1p2`.







# References
1. https://learning.lpi.org/en/learning-materials/101-500/101/101.1/101.1_01/
2. 
