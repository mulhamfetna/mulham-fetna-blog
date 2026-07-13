---
title: Dual-Boot System Recovery & Maintenance Guide
date: 2026-03-31
description: "A practical guide to UEFI boot mechanics, dual-boot failure modes, and recovering a broken Linux/Windows setup."
keywords: ["dual boot recovery", "UEFI boot repair", "Linux Windows dual boot", "GRUB recovery"]

---


# Comprehensive Dual-Boot System Recovery & Maintenance Guide
## Arch Linux & Windows 11 UEFI Systems

---

## 1. System Architecture & Failure Modes

### 1.1 UEFI Boot Mechanics
Modern dual-boot systems rely on UEFI NVRAM (Non-Volatile Random Access Memory) to store boot entries. Unlike Legacy BIOS (MBR), UEFI maintains a database of bootloaders independent of disk order. Common failure modes include:

- **NVRAM Wipe**: BIOS reset, SSD removal/reinstallation, or CMOS battery loss clears boot entries
- **EFI Partition Misalignment**: Moving/resizing partitions changes start sectors, breaking GRUB's file references
- **Filesystem Locks**: Windows Fast Startup/hibernation marks NTFS as dirty, preventing Linux write access
- **Kernel Absence**: Incomplete mounts or partition moves resulting in missing `/boot/vmlinuz-linux`

### 1.2 Pre-Recovery Data Collection
Before manipulation, capture system state:

```bash
# Boot from Arch Live ISO
sudo fdisk -l /dev/sda          # Partition table geometry
sudo lsblk -f                   # Filesystem types and UUIDs
sudo blkid                      # Detailed partition attributes
efibootmgr -v                   # Current UEFI boot entries (if available)
```

---

## 2. Bootloader Recovery & GRUB Restoration

### 2.1 Scenario A: Lost Boot Entries (Post-BIOS Reset)
**Symptoms**: System boots directly to Windows; no GRUB menu; missing Linux entry in firmware boot menu.

**Root Cause**: UEFI NVRAM entries deleted while EFI partitions remain intact.

**Recovery Procedure**:

1. **Boot Environment Preparation**
   ```bash
   # Identify partitions (adjust device identifiers as needed)
   sudo mount /dev/sda7 /mnt          # Arch root partition
   sudo mount /dev/sda6 /mnt/boot/efi # Arch EFI partition
   sudo mount /dev/sda8 /mnt/home     # Optional: home partition
   
   # Virtual filesystem binding for chroot
   sudo mount --bind /dev /mnt/dev
   sudo mount --bind /proc /mnt/proc
   sudo mount --bind /sys /mnt/sys
   ```

2. **Chroot Entry**
   ```bash
   sudo arch-chroot /mnt
   # Alternatively, manual method:
   # chroot /mnt /bin/bash
   ```

3. **GRUB Reinstallation**
   ```bash
   # Re-register GRUB with UEFI firmware
   grub-install --target=x86_64-efi \
                --efi-directory=/boot/efi \
                --bootloader-id=GRUB \
                --recheck
   
   # Regenerate configuration with OS-prober for Windows detection
   grub-mkconfig -o /boot/grub/grub.cfg
   ```

4. **Boot Priority Restoration**
   ```bash
   # Verify entry creation
   efibootmgr -v
   
   # Set GRUB as first boot option (replace XXXX with boot number)
   efibootmgr -o XXXX,YYYY,ZZZZ
   ```

### 2.2 Scenario B: GRUB Rescue Prompt (Post-Partition Move)
**Symptoms**: `grub rescue>` prompt; "unknown filesystem" errors; partition resized/moved recently.

**Root Cause**: GRUB's core.img references incorrect block addresses after partition geometry changes.

**Extended Recovery**:

1. **Manual Boot (Temporary)**
   ```grub
   # Identify partition containing /boot
   ls (hd0,gpt6)/
   
   # Set root and prefix
   set root=(hd0,gpt6)
   set prefix=(hd0,gpt6)/boot/grub
   insmod normal
   normal
   ```

2. **Complete Restoration via Live ISO**
   ```bash
   # Mount partitions
   sudo mount /dev/sda6 /mnt          # Root
   sudo mount /dev/sda5 /mnt/boot/efi # EFI (new location after move)
   
   sudo arch-chroot /mnt
   
   # Critical: Update fstab for new UUIDs if partitions were reformatted
   nano /etc/fstab
   
   # Reinstall GRUB to new EFI location
   grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=GRUB
   
   # Kernel restoration (essential if /boot was unmounted during moves)
   pacman -S linux linux-lts          # Reinstall kernel packages
   mkinitcpio -P                      # Regenerate initramfs for all kernels
   
   # Final configuration
   grub-mkconfig -o /boot/grub/grub.cfg
   ```

3. **UEFI Entry Cleanup**
   ```bash
   # Remove stale entries (optional)
   efibootmgr -b XXXX -B              # Delete old entry by number
   
   # Create new entry manually if automated creation fails
   efibootmgr -c -d /dev/sda -p 5 -l /EFI/GRUB/grubx64.efi -L "GRUB"
   ```

### 2.3 BIOS Configuration Requirements
Access firmware settings (F2/Del during POST) and verify:
- **Boot Mode**: UEFI only (disable Legacy/CSM)
- **Secure Boot**: Disabled (unless using signed kernels/shim)
- **SATA Mode**: AHCI (not RAID/Intel RST)
- **Fast Boot**: Disabled for troubleshooting (prevents USB initialization)

---

## 3. Permanent Filesystem Configuration

### 3.1 NTFS Data Partition Integration
**Objective**: Mount Windows data partitions at boot with full read-write access for regular users.

**Implementation**:

1. **Prerequisites**
   ```bash
   sudo pacman -S ntfs-3g fuse2       # FUSE-based NTFS driver
   ```

2. **UUID Identification**
   ```bash
   sudo blkid /dev/sda3
   # Output: UUID="01DC1F4F8BC938D0" TYPE="ntfs" LABEL="DATA"
   ```

3. **Mount Point Creation**
   ```bash
   sudo mkdir -p /mnt/data
   sudo chown $USER:$USER /mnt/data   # Pre-set ownership
   ```

4. **fstab Configuration**
   ```bash
   sudo cp /etc/fstab /etc/fstab.bak  # Backup
   
   # Edit configuration
   sudo nano /etc/fstab
   
   # Add entry:
   UUID=01DC1F4F8BC938D0  /mnt/data  ntfs-3g  defaults,uid=1000,gid=1000,umask=000,noatime,nofail  0  0
   ```

   **Parameter Analysis**:
   - `uid=1000,gid=1000`: Maps all files to your user/group ID (verify with `id` command)
   - `umask=000`: Full permissions (rwX) for owner; adjust to `umask=022` for read-only others
   - `noatime`: Prevents write amplification on SSDs (NTFS journal overhead)
   - `nofail`: Prevents boot failure if drive is missing/disconnected

5. **Activation**
   ```bash
   sudo systemctl daemon-reload
   sudo mount -a
   findmnt --verify                   # Validate syntax
   ```

### 3.2 Alternative: NTFS3 Kernel Driver (Modern Approach)
For kernel 5.15+, the native `ntfs3` driver offers improved performance:

```bash
# In /etc/fstab:
UUID=XXXX  /mnt/data  ntfs3  uid=1000,gid=1000,umask=000,nofail  0  0
```

**Note**: `ntfs3` lacks NTFS compression support; use `ntfs-3g` if compressing files from Linux.

---

## 4. Cross-Platform Filesystem Troubleshooting

### 4.1 NTFS Lock States (Windows Hibernation)
**Symptoms**: Mount error "The disk contains an unclean file system"; read-only mount despite fstab settings; `Operation not permitted` on write.

**Mechanism**: Windows Fast Startup creates `hiberfil.sys`, marking the NTFS journal as in-use.

**Resolution Hierarchy**:

#### Method 1: Windows-side Permanent Fix (Recommended)
Disable Fast Startup in Windows:
- **Control Panel** → **Power Options** → **Choose what power buttons do** → **Change settings unavailable** → Uncheck **Turn on fast startup**

#### Method 2: Linux Force Mount (Data Risk)
```bash
# Removes hibernation file (destroys unsaved Windows session)
sudo mount -t ntfs-3g -o remove_hiberfile /dev/sda3 /mnt/data
```

#### Method 3: Repair Utilities
```bash
# Clear dirty bit (limited effectiveness)
sudo ntfsfix /dev/sda3

# Full Windows repair (requires Windows PE/Recovery)
# Boot Windows Recovery → Command Prompt
chkdsk D: /f /r /x
# /f: Fix errors | /r: Recover bad sectors | /x: Force dismount
```

### 4.2 Busy Mount Resolution
When unmount fails with "target is busy":

```bash
# Identify blocking processes
sudo fuser -m /mnt/data
sudo lsof | grep /mnt/data

# Graceful termination
sudo kill -15 <PID>

# Force unmount (lazy detach)
sudo umount -l /mnt/data
```

### 4.3 Partition Recovery (Advanced)
If partition table corruption occurs during resizing:

```bash
# Install recovery tools
sudo pacman -S testdisk

# Interactive recovery
sudo testdisk /dev/sda
# Select: Intel/PC → Analyze → Quick Search → Deeper Search → Write
```

---

## 5. Diagnostic Reference Matrix

| Symptom | Diagnostic Command | Root Cause | Solution |
|---------|-------------------|------------|----------|
| GRUB rescue prompt | `ls` in GRUB shell | Moved/deleted EFI partition | Section 2.2 |
| Boots directly to Windows | `efibootmgr -v` | NVRAM entry lost | Section 2.1 |
| `mount: unknown filesystem type 'ntfs'` | `lsblk -f` | Missing `ntfs-3g` | Section 3.1 |
| NTFS read-only | `dmesg \| grep ntfs` | Windows hibernation lock | Section 4.1 |
| Missing kernel in GRUB | `ls /boot` in chroot | Unmounted `/boot` during install | Section 2.2 (kernel reinstall) |
| `failed to mount /boot/efi` | `blkid` vs `/etc/fstab` | UUID mismatch after format | Update fstab UUIDs |

---

## 6. Validation Checklist

Post-recovery verification steps:

- [ ] `efibootmgr -v` shows GRUB entry with correct path `\EFI\GRUB\grubx64.efi`
- [ ] `sudo grub-mkconfig` detects both Linux and Windows bootloaders
- [ ] Reboot test: System presents GRUB menu (timeout ≥ 5 seconds)
- [ ] Windows boot successful, then Linux boot successful (bi-directional)
- [ ] `findmnt /mnt/data` shows NTFS partition with `rw` flags
- [ ] Test write: `touch /mnt/data/linux_test_file` succeeds without sudo
- [ ] Windows restart does not re-lock NTFS (Fast Startup disabled)

---

## 7. Prevention & Maintenance

1. **UEFI Entry Backup**
   ```bash
   sudo efibootmgr -v > /boot/efi/efi_backup_$(date +%Y%m%d).txt
   ```

2. **fstab Immutability**
   After confirmed working configuration:
   ```bash
   sudo chattr +i /etc/fstab  # Prevent accidental modification
   sudo chattr -i /etc/fstab  # When changes needed
   ```

3. **Pre-Resize Protocol**
   Before partition manipulation:
   - Disable Windows Fast Startup
   - Disable BIOS Secure Boot
   - Create GRUB rescue USB with `grub-install --removable`

4. **Monitoring**
   ```bash
   # Weekly filesystem check
   sudo ntfsfix -n /dev/sda3  # No-action check for dirty bit
   ```

---

This guide consolidates bootloader recovery, permanent mounting configuration, and cross-platform filesystem troubleshooting into a single reference for maintaining complex dual-boot environments. All procedures assume UEFI/GPT partitioning; adapt device identifiers (`sda`, `nvme0n1`, etc.) to your specific hardware topology.

---

## 8. Frequently Asked Questions (FAQ)

### 8.1 UEFI Boot Architecture

**Q1: Why does removing the SSD physically clear UEFI boot entries when the data is still intact?**
A: UEFI stores boot entries in NVRAM (non-volatile memory) on the motherboard, not on the disk. These entries contain pointers to EFI files (like `\EFI\GRUB\grubx64.efi`). When you remove the SSD, the firmware detects the missing device and often purges invalid entries during POST. Additionally, resetting BIOS/CMOS clears NVRAM entirely. The EFI partition itself remains unharmed, but the firmware "forgets" the path to the bootloader.

**Q2: What is the fundamental difference between Legacy BIOS (MBR) and UEFI boot modes?**
A: Legacy BIOS uses the Master Boot Record (first 512 bytes of disk) containing stage-1 boot code that chainloads stage-2 from disk sectors. UEFI uses a FAT32-formatted EFI System Partition (ESP) containing `.efi` executable files. UEFI maintains a boot manager database in NVRAM, supports GPT partitioning (no 2TB limit), and offers Secure Boot capabilities. Legacy modifies disk sectors directly; UEFI launches applications from the filesystem.

**Q3: Can I use a single EFI partition for both Windows and Linux?**
A: Yes, but it is not recommended for complex setups. Both Windows and Linux can coexist in `/boot/efi/EFI/Microsoft` and `/boot/efi/EFI/GRUB` respectively. However, Windows Update occasionally "cleans" the EFI partition and may remove non-Microsoft bootloaders. Separate EFI partitions (Windows on sda1, Linux on sda6) provide isolation but require manual boot entry management via `efibootmgr`.

**Q4: Why does GRUB rescue appear immediately after I resized/moved partitions with GParted?**
A: GRUB's core.img contains absolute block addresses (not filesystem paths) for stage-2 files. When you move a partition, the start sector changes, but GRUB's embedded pointer still references the old physical location. The rescue prompt appears because GRUB cannot locate its normal.mod or configuration files at the expected disk offsets.

**Q5: How can I manually boot my system from the GRUB rescue prompt without a live USB?**
A: You can manually chainload if you know the partition layout:
```grub
grub rescue> ls (hd0,gpt6)/           # List root contents
grub rescue> set root=(hd0,gpt6)
grub rescue> set prefix=(hd0,gpt6)/boot/grub
grub rescue> insmod normal
grub rescue> normal
grub rescue> insmod linux
grub rescue> linux /boot/vmlinuz-linux root=/dev/sda6
grub rescue> initrd /boot/initramfs-linux.img
grub rescue> boot
```
This temporarily boots the system so you can permanently reinstall GRUB.

### 8.2 Chroot & System Recovery

**Q6: What is the difference between `chroot` and `arch-chroot`, and when must I use the manual method?**
A: `arch-chroot` is a wrapper that automatically mounts `/dev`, `/proc`, `/sys`, `/run`, and binds `/etc/resolv.conf` before entering the chroot. Use manual `chroot` when `arch-chroot` fails or on non-Arch systems:
```bash
mount --types proc /proc /mnt/proc
mount --rbind /sys /mnt/sys
mount --make-rslave /mnt/sys
mount --rbind /dev /mnt/dev
mount --make-rslave /mnt/dev
mount --bind /run /mnt/run
mount --make-slave /mnt/run
cp /etc/resolv.conf /mnt/etc/
chroot /mnt /bin/bash
```

**Q7: Why does `grub-install` fail with "EFI variables cannot be set on this system"?**
A: This occurs when booted in Legacy BIOS mode or when the EFI filesystem drivers are not loaded. Inside chroot, you must bind-mount the efivarfs:
```bash
mount -t efivarfs efivarfs /sys/firmware/efi/efivars
```
Also ensure you're booted the USB in UEFI mode (not "Legacy USB Support").

**Q8: What happens if my `/boot` directory was unmounted during a kernel upgrade, and how do I detect this?**
A: The kernel image (`vmlinuz-linux`) and initramfs are written to the root filesystem's `/boot` directory instead of the EFI partition. GRUB configuration points to the EFI partition, so it cannot find the kernel. Symptoms: GRUB menu shows only Windows. Detection:
```bash
# In chroot
ls -la /boot/efi/EFI/GRUB/grubx64.efi  # Should exist
ls -la /boot/vmlinuz-linux             # If missing here but pacman says installed, /boot wasn't mounted
```
Fix: Mount `/boot` correctly, reinstall kernel packages `pacman -S linux`, regenerate GRUB config.

**Q9: How do I recover if I accidentally formatted my Linux EFI partition?**
A: Boot live ISO, mount root and home (but not the missing EFI). Create new FAT32 partition, mount it at `/mnt/boot/efi`, then:
```bash
arch-chroot /mnt
pacman -S grub efibootmgr
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=GRUB
grub-mkconfig -o /boot/grub/grub.cfg
```
You will need to manually copy `grubx64.efi` to the new partition; GRUB reinstall handles this.

**Q10: Why does `mkinitcpio -P` fail with "command not found" inside chroot?**
A: The chroot environment lacks the PATH variable or base-devel tools. Export PATH explicitly:
```bash
export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```
Or ensure `base` and `mkinitcpio` packages are installed: `pacman -S base mkinitcpio linux`.

### 8.3 Filesystem & fstab Configuration

**Q11: How do I determine my actual user UID and GID for fstab configuration?**
A: Run `id` in terminal:
```
uid=1000(username) gid=1000(username) groups=1000(username),3(sys),90(network),98(power)
```
The first number (1000) is your UID. For system consistency, use `1000` for the primary user. If you have multiple users, create a shared group or use `gid=users` (typically 100).

**Q12: Why is my NTFS partition mounted read-only despite using ntfs-3g and correct fstab options?**
A: Three common causes:
1. **Windows hibernation**: See Section 4.1. Check with `ntfs-3g.secaudit /dev/sda3 | grep hibernated`
2. **Filesystem errors**: NTFS marked dirty. Run `sudo ntfsfix /dev/sda3` or Windows chkdsk
3. **Mount precedence**: An earlier mount (systemd, udisks) mounted it read-only before fstab applied. Check `findmnt /mnt/data` and unmount before `mount -a`

**Q13: Should I use `ntfs-3g` (FUSE) or the newer `ntfs3` (kernel driver)?**
A: Use `ntfs-3g` if you need:
- Full POSIX permission support (chmod/chown)
- Windows compression (NTFS-LZNT1) support
- Proven stability for critical writes

Use `ntfs3` (kernel 5.15+) if you want:
- Better performance (native kernel vs FUSE overhead)
- Lower CPU usage
- Simple mount options (no `uid=/gid=` needed if using ACLs)

**Q14: What are the risks of using `remove_hiberfile` mount option on a production Windows system?**
A: This deletes `hiberfil.sys`, destroying:
- Unsaved work in hibernated Windows sessions
- Windows Fast Startup cache (causing slower subsequent boots)
- Hybrid sleep data
Always attempt Windows full shutdown (Shift+Shutdown) or `powercfg /hibernate off` from Windows first.

**Q15: Why does `mount -a` report "can't find UUID=XXXX" when `blkid` shows the UUID exists?**
A: This indicates the kernel hasn't updated its partition table cache after a disk modification. Run:
```bash
partprobe /dev/sda
# or
blockdev --rereadpt /dev/sda
```
If the device is busy, you may need to reboot or use `partprobe -s`.

### 8.4 Cross-Platform Issues

**Q16: Can I disable Windows Fast Startup from within Linux without booting Windows?**
A: No. Fast Startup is a Windows registry and powercfg setting. However, you can force a "full shutdown" from Windows command line (if you can access Windows RE):
```cmd
shutdown /s /f /t 0
```
The only Linux-side workaround is `remove_hiberfile` (destructive) or mounting read-only.

**Q17: Why does Windows CHKDSK run automatically every time I boot Windows after using Linux?**
A: Windows detects the "dirty bit" set by `ntfsfix` or forced mounts. While `ntfsfix` clears some flags, it may set others that Windows interprets as "volume modified by external OS." Allow CHKDSK to run once; it will reset internal consistency markers.

**Q18: Is it safe to resize NTFS partitions from Linux using GParted?**
A: GParted uses `ntfsresize` (part of `ntfs-3g` suite) which is generally safe, but:
- Always defragment Windows first (`defrag C: /O`)
- Disable Windows pagefile and hibernation during resize
- Never resize a "dirty" NTFS partition (run chkdsk first)
- Have Windows repair media ready (rarely, Windows may need to reassign drive letters)

**Q19: How do I access my Linux ext4 partition from Windows for data recovery?**
A: Use Windows Subsystem for Linux (WSL2) with `wsl --mount --type ext4`, or install third-party drivers like Ext2Fsd (discontinued, risky) or Paragon ExtFS. For recovery, boot Linux live ISO instead—native Linux tools are safer than Windows drivers for ext4.

**Q20: Can I use `dd` to clone my dual-boot disk to a new SSD?**
A: Yes, but post-clone you must:
```bash
# On the new disk
sgdisk -G /dev/sdb  # Randomize GUID to avoid conflicts
efibootmgr -c -d /dev/sdb -p 1 -L "GRUB Clone" -l '\EFI\GRUB\grubx64.efi'
```
Clone with `dd if=/dev/sda of=/dev/sdb bs=4M status=progress` or use `ddrescue` for failing disks.

### 8.5 Advanced Troubleshooting

**Q21: What is Secure Boot, why does it prevent GRUB installation, and can I use it with Arch?**
A: Secure Boot verifies cryptographic signatures on EFI binaries against Microsoft's UEFI CA. Unsigned GRUB (default Arch) is rejected. Solutions:
- Disable Secure Boot (easiest)
- Use `shim` and `preloader` to chainload unsigned GRUB (complex)
- Sign your own kernels and GRUB with MOK (Machine Owner Key) using `sbsigntools`

**Q22: How do I create a Windows 10/11 bootable USB from Arch Linux for repairing BCD/CHKDSK?**
A: Use `woeusb` or manual formatting:
```bash
# Replace /dev/sdX with your USB (not partition)
sudo wipefs -a /dev/sdX
sudo fdisk /dev/sdX  # Create GPT, partition 1 type Microsoft basic data
sudo mkfs.fat -F32 /dev/sdX1
sudo mkdir -p /mnt/winusb
sudo mount /dev/sdX1 /mnt/winusb
# Mount Windows ISO
sudo mount -o loop win11.iso /mnt/iso
sudo cp -r /mnt/iso/* /mnt/winusb/
sudo sync
```
Alternatively, use Ventoy for multi-ISO USB.

**Q23: Why does my system boot to a black screen after selecting Arch in GRUB?**
A: Common causes:
- **Missing GPU drivers**: If you installed NVIDIA/AMD drivers but boot generic kernel without initramfs updates
- **Incorrect root UUID**: Kernel parameter `root=` points to wrong partition (check `/etc/default/grub` and `grub.cfg`)
- **Fast Boot enabled in BIOS**: GPU not initialized properly; disable in UEFI settings

**Q24: How do I restore Windows Boot Manager as the default bootloader temporarily?**
A: From Windows:
```cmd
bcdedit /set {bootmgr} path \EFI\Microsoft\Boot\bootmgfw.efi
```
From Linux:
```bash
efibootmgr -n 000X  # Set Windows entry as next boot only
# or permanently:
efibootmgr -o 000X,000Y  # Windows first, GRUB second
```

**Q25: What is the difference between UUID and PARTUUID, and which should I use in fstab?**
A: 
- **UUID**: Filesystem identifier (e.g., `01DC1F4F8BC938D0`). Changes when you reformat the partition. Use for data partitions.
- **PARTUUID**: GPT partition table entry identifier (e.g., `12345678-1234-1234-1234-123456789abc`). Persists across formats. Use for root/boot in `/etc/fstab` to avoid boot failures after accidental reformats.
Syntax: `PARTUUID=xxxx-yyyy` vs `UUID=xxxx`.

**Q26: How do I backup and restore my GPT partition table (not the data)?**
A: Backup:
```bash
sgdisk -b gpt_backup.bin /dev/sda
```
Restore (destructive):
```bash
sgdisk -l gpt_backup.bin /dev/sda
partprobe /dev/sda
```
Store `gpt_backup.bin` on external media. This preserves partition layout but not filesystem contents.

**Q27: Why does `findmnt --verify` warn about "ntfs-3g: unknown filesystem type"?**
A: The verification tool checks kernel filesystem support, but `ntfs-3g` is a FUSE module, not a kernel driver. This warning is cosmetic; the mount will succeed if `ntfs-3g` is installed. For kernel-native `ntfs3`, the warning disappears.

**Q28: Can I convert my existing Legacy BIOS install to UEFI without reinstalling?**
A: Yes, if your motherboard supports UEFI:
1. Boot live ISO in UEFI mode
2. Create FAT32 EFI partition (~512MB)
3. Mount root and EFI
4. `arch-chroot /mnt`
5. `pacman -S grub efibootmgr`
6. `grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=GRUB`
7. Update fstab with new EFI UUID
8. Reboot and switch BIOS to UEFI mode

**Q29: What should I do if `arch-chroot` fails with "failed to run command '/bin/bash': No such file or directory"?**
A: This indicates the root partition was mounted without the actual system files (possibly mounted the EFI partition as root by mistake), or the bash binary is corrupted. Verify:
```bash
ls /mnt/bin/bash
# If missing, you mounted wrong partition. If exists but error persists:
mount -t proc proc /mnt/proc
mount -t sysfs sys /mnt/sys
mount -o bind /dev /mnt/dev
# Then try chroot with explicit path:
chroot /mnt /usr/bin/bash
```

**Q30: How do I handle a system where both OSes show in GRUB, but Windows crashes with "INACCESSIBLE_BOOT_DEVICE" after I fixed Linux?**
A: This occurs when GRUB's chainloader passes incorrect firmware parameters to Windows, or when the Windows EFI partition was accidentally modified. Fix from Windows Recovery:
```cmd
bootrec /fixboot
bootrec /scanos
bootrec /rebuildbcd
```
Or from Linux, ensure `os-prober` correctly identifies Windows EFI partition:
```bash
pacman -S os-prober
grub-mkconfig -o /boot/grub/grub.cfg
```

---

## 9. Official System Documentation & References

### 9.1 Arch Linux Ecosystem

**Arch Wiki** (Primary Authority)
- **Dual boot with Windows**: https://wiki.archlinux.org/title/Dual_boot_with_Windows
- **GRUB**: https://wiki.archlinux.org/title/GRUB
- **Fstab**: https://wiki.archlinux.org/title/Fstab
- **NTFS-3G**: https://wiki.archlinux.org/title/NTFS-3G
- **NTFS (Kernel Driver)**: https://wiki.archlinux.org/title/NTFS
- **Chroot**: https://wiki.archlinux.org/title/Chroot
- **Unified Extensible Firmware Interface**: https://wiki.archlinux.org/title/Unified_Extensible_Firmware_Interface
- **Partitioning**: https://wiki.archlinux.org/title/Partitioning
- **System Recovery**: https://wiki.archlinux.org/title/Installation_guide#Chroot

**Arch Manual Pages**
- `man 5 fstab` – Static filesystem information
- `man 8 mount` – Mount filesystems
- `man 8 ntfs-3g` – NTFS FUSE driver manual
- `man 8 grub-install` – Install GRUB on a device
- `man 8 efibootmgr` – Manage UEFI Boot Entries
- `man 8 mkinitcpio` – Create initial ramdisk environments

### 9.2 GNU & Linux Kernel Documentation

**GNU GRUB Manual**
- **UEFI Installation**: https://www.gnu.org/software/grub/manual/grub/html_node/Installing-GRUB-using-grub_002dinstall.html
- **GRUB Rescue**: https://www.gnu.org/software/grub/manual/grub/html_node/GRUB-only-offers-a-rescue-shell.html

**Linux Kernel Documentation**
- **NTFS3 Driver** (Kernel 5.15+): https://www.kernel.org/doc/html/latest/filesystems/ntfs3.html
- **EFI Variables**: https://www.kernel.org/doc/html/latest/admin-guide/efi.html
- **Filesystem Hierarchy Standard**: https://refspecs.linuxfoundation.org/FHS_3.0/fhs/index.html

### 9.3 Microsoft Technical Documentation

**Windows Hardware Developer Center**
- **UEFI Firmware**: https://docs.microsoft.com/en-us/windows-hardware/drivers/bringup/uefi-firmware
- **Boot Configuration Data (BCD)**: https://docs.microsoft.com/en-us/windows-hardware/manufacture/desktop/bcd-boot-options-reference
- **CHKDSK Command**: https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/chkdsk
- **Fast Startup Technical Details**: https://docs.microsoft.com/en-us/windows-hardware/design/device-experiences/powercfg-command-line-options

**Windows Recovery Environment (WinRE)**
- **Bootrec.exe Tool**: https://docs.microsoft.com/en-us/windows-hardware/manufacture/desktop/bootsect-command-line-options
- **BCDEdit**: https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/bcdedit

### 9.4 UEFI Specifications

**UEFI Forum**
- **UEFI Specification 2.9B** (PDF): https://uefi.org/specifications
- **Secure Boot**: https://uefi.org/revocationlistfile
- **GPT Partitioning**: Chapter 5 of UEFI Specification (GUID Partition Table)

### 9.5 Filesystem & Recovery Tools

**NTFS-3G Project**
- **Documentation**: https://github.com/tuxera/ntfs-3g/wiki
- **Advanced Options**: https://www.tuxera.com/community/open-source-ntfs-3g/

**GParted**
- **Resizing NTFS**: https://gparted.org/features.php
- **Documentation**: https://gparted.org/display-doc.php

**TestDisk & PhotoRec**
- **NTFS Repair Guide**: https://www.cgsecurity.org/wiki/NTFS_Boot_sector_recovery
- **Partition Table Recovery**: https://www.cgsecurity.org/wiki/TestDisk

### 9.6 Dell Precision-Specific Resources

**Dell Support**
- **Precision 3551 UEFI Configuration**: https://www.dell.com/support/manuals/precision-3551-workstation
- **BIOS Setup Guide**: https://www.dell.com/support/kbdoc/000124211

---

## 10. Version History & Change Log

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2026-03-31 | Initial comprehensive guide combining bootloader recovery, NTFS mounting, and UEFI troubleshooting |

---

**Disclaimer**: This guide involves low-level disk and firmware operations. Always backup critical data before executing partition table modifications, bootloader reinstallations, or filesystem repairs. The procedures assume x86_64 architecture with GPT partitioning; adaptations may be required for ARM64 or MBR systems.