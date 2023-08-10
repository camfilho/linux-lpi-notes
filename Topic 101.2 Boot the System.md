The kernel must b loaded by a program called a bootloader, which itself is loaded by a pre-installed firmware. BIOS or UEFI
The bootloader can be customized to pass parameters to the kernel.
- e.g. which partition contains the root filesystem
- e.g which mode the operating system should execute.

The kernel, after loaded, continues the boot process identifying and configuring the hardware. The kernel calls the utility responsible for starting and managing the system's services.


## BIOS or UEFI

### BIOS
- The BIOS is stored in a non-volatile memory chop attached to the motherboard, executed every time the computer is powered on.
- This type of program is called firmware its storage location is separate from the other storage devices. The first 440 byes in the first storage
- The first 440 bytes in the first storage device -following the order defined in the BIOS configuration utility - are the first stage of the bootloader (also called bootstrap).
- The first 512 bytes  of the storage are named the MBR (Master Boot Record), OF STORAGE DEVICES USING THE STANDARD DOS partition schema, and in addition to the first stage of the bootloader, contain the partition table.
- If the MBR does contain the correct data, the system will not be able to boot, unless an alternative method is employed.

BIOS pre-operating steps:

- POST process: Hardware check during startup to detect basic failures.
- BIOS activation: Enables essential components like video, keyboard, and storage media.
- Bootloader loading: BIOS loads the first stage of the bootloader from MBR.
- Second stage bootloader: Responsible for presenting boot options and loading the system.

### UEFI

- UEFI - Unified Extensible Firmware Interface.
- UEFI is also a firmware
- It can identify partitions and read may filesystems found in them.
- UEFI does not rely on the MBR. It takes into account only the settings stored in its non-volatile memory (NVRAM) attached to the motherboard.
- The settings stores the definitions that indicate the location of the UEFI compatible programs. EFI APplications.
- EFI Applications can be bootloaders, operating system selectors, tools for system diagnostics and repair.
- The partition containing the EFI applications is called - EFI System Partition - **ESP**

THe Pre-operating system boot steps on a UEFI system:

1. POST process: Checks for hardware failures upon powering on the machine.
2. UEFI activation: Enables basic components for system loading, including video, keyboard, and storage.
3. UEFI firmware execution: Reads NVRAM definitions to run the pre-defined EFI application from ESP partition, often a bootloader.
4. Bootloader loading: If the pre-defined EFI application is a bootloader, it loads the kernel to initiate the operating system.

## The Bootloader

- Sometimes the list(list of OS available to boot) does no appear automatically.
- Press SHIFT while GRUB is being called by BIOS.
- Press ESC while GRUBD is being called by UEFI

Some of the most useful kernel parameters are:
1. acpi - Enable/Disables ACPI support. `acpi=off`
2. init - `init=/bin/bash` sets the Bash shell as the initiator
3. systemd.unit - Sets the systemd target to be activated. `systemd.unit=graphical.target`. it also accepts SysV runlevels
4. mem - Sets the amount of available RAM for the system. Useful for virtual machines.  `mem=512M` limits the available ram memory to 512 Megabytes.
5. maxcpus - Limits the number of processor cores visible to the system. 0 turns off the support for multi-processor machines.
6. quite - Hides boot messages
7. vga - Selects video mode. `vga=ask` shows list of the available modes to choose.
8. root - sets the root partition, distinct from the one pre-configured in the bootloader. `root=/dev/sda3`
9. rootflags - mount options for the root filesystem
10. ro - makes root filesystem read only
11. rw - allows writing in the RO FS during initial mount


- Kernel parameters must be added to the file `/etc/default/grub` in the line `GRUB_CMDLINE_LINUX` to make them persistent across reboots.
- A new configuration file for the bootloader must be generated every time `/etc/default/grub` changes.
- `grub-mkconfig -o /boot/grub/grub.cfg` to generate the configuration file

## System Initialization
- Services - `deamons` may be active all the time.

### The Operating System
- Strictly speaking, the operating system is just the kernel and its components which control the hardware and manages all processes.

- The initialization of the operating system starts when the bootloader loads the kernel into RAM.
- The kernel then takes charge of the CPU, detect and setup the fundamental aspects of the operating system such as basic hardware configuration and memory addressing.
- The kernel will then open the `initramfs`
- The initramfs filesystem used as a temporary root filesystem during the boot process.
- Main purpose of the `initramfs`  file is to provide the requred modules, so the kernel can access the "real" root filesystem of the operating system.

- As soon as the root filesystem is available, the kernel will mount all the filesystems configured in `/etc/fstab` and then execute the first program called `init`.
- Init is responsible for running all initialization scripts and system daemons
## SysV standard
- runlevels
### systemd
- Systemd is a modern system and services manager with a compatibility layer for the SysV command ans runlevels. Systemd has a concurrent structure, employs sockets and D-Bus for service activation, on-demand daemon execution, process monitoring with cgroups, snapshot support, system session recovery, mount point control, dependency-based service control.
- ### Upstart
Like `systemd`, Upstart is a substitute to init.

## Initialization Inspection
The memory space where the kernel stores its messages, including the boot messages, is called `KERNEL RING BUFFER`.
- **dmesg** displays the current messages in the kernel ring buffer.

```bash
$ dmesg
[ 5.262389] EXT4-fs (sda1): mounted filesystem with ordered data mode. Opts:
(null)
[ 5.449712] ip_tables: (C) 2000-2006 Netfilter Core Team
[ 5.460286] systemd[1]: systemd 237 running in system mode.
[ 5.480138] systemd[1]: Detected architecture x86-64
```
The values in the beginning of the lines are the amount of seconds relative to when the kernel load begins.

- In systems based on systemd, `journalctl` will show the initialization messages with `-b, --boot, -k or --dmesg` to how the ring buffer as dmsg

- Initialization and other messages issued by the operating system are stored in files inside the directory `/var/log/`


































# References
1. 