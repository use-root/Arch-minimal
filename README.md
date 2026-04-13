# Minimal Arch Linux on Oracle VirtualBox ௹

This is my workflow for hacking. Previously, I used `Kali-minimal`, and honestly, the differences are minimal.


![BSPWM](./Entorno.jpg)

### Minimal recommendations:
```javascript
RAM (Base Memory): 4 GB
Disk: 50 GB
Processors: 3
Acceleration: Nested Paging, KVM Paravirtualization (If you are on Linux, you can use QEMU + KVM — it is much better).
Video Memory: 75 MB
```

### Disk partitioning:

I use the following structure:

- `/boot` or `/boot/efi` (≤ 512 MB)
- `/` (root) (30 GB)
- `/home` (45 GB)
- `swap` (4–8 GB depending on RAM)

For Arch, I use `cfdisk` and select GPT. Even in a VM, GPT is more modern than DOS, but you can choose either.

After creating the partitions, format them:

```bash
mkfs.vfat -F 32 /dev/sda1   # EFI
mkfs.ext4 /dev/sda2         # root
mkfs.ext4 /dev/sda3         # home
mkswap /dev/sda4
swapon /dev/sda4
```

Partition layout:

| Partition | Filesystem | Size  |
|----------|-----------|------|
| /boot    | FAT32 (EFI) | 512 MB |
| /        | ext4      | 30 GB |
| /home    | ext4      | 45 GB |
| swap     | swap      | 4 GB  |

### Mount partitions:

```bash
mkdir -p /mnt/{boot,home}
mount /dev/sda2 /mnt
mount /dev/sda1 /mnt/boot
mount /dev/sda3 /mnt/home

pacstrap -K /mnt linux linux-firmware base base-devel grub efibootmgr wpa_supplicant networkmanager
```

### Package explanation:

- **linux:** The Linux kernel.
- **linux-firmware:** Firmware for GPU, network, sound, etc.
- **networkmanager:** Manages network connections (Wi-Fi, Ethernet, VPN).
- **wpa_supplicant:** Handles WPA/WPA2 Wi-Fi authentication.
- **grub:** Bootloader that loads the kernel.
- **efibootmgr:** Registers the boot entry in UEFI firmware.
- **base / base-devel:** Essential tools (bash, grep, make, gcc, etc.), useful for building software and AUR.

---

### FSTAB (important):

`fstab` is a configuration file that tells the system **which partitions to mount automatically at boot and where**.

You generate it with:

```bash
genfstab -U /mnt >> /mnt/etc/fstab
```

What it does:
- Detects your partitions
- Uses UUIDs (safer than `/dev/sdX`)
- Defines mount points (`/`, `/home`, `/boot`, etc.)

Example:

```bash
UUID=xxxx-xxxx / ext4 defaults 0 1
UUID=yyyy-yyyy /home ext4 defaults 0 2
UUID=zzzz-zzzz /boot vfat defaults 0 2
```

Without `fstab`, your system will not know how to mount disks on boot.

---

### VirtualBox tip:

Enable 3D acceleration (recommended).

---

### Tiling Window Manager:

BSPWM + SXHKD

- Terminal: Alacritty
- Picom
- Neovim
- bat (alias for cat)
- lsd (alias for ls)
- powerlevel10k (Zsh theme)
- Shell: zsh
- rofi (launcher)
- VirtualBox Guest: VBoxClient --clipboard
