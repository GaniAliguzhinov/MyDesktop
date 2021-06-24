# MyDesktop

## Set up live usb

```sh
lsblk
sudo dd if=arch.img of=/dev/sdb status=progress bs=1M; sync
```

## Boot

## Install

```sh
ls /sys/firmware/efi/efivars # verify boot mode
```

Now usb tether

```sh
ping archlinux.org
timedatectl set-ntp true # sys clock
timedatectl set-timezone Asia/Almaty
timedatectl status
```

### Partition

```sh
fdisk -l
```

Shrink existing

```sh
e2fsck -f /dev/nvme0n1p6 # check ext4 fs
resize2fs /dev/nvme0n1p6 120G # actually resize fs

parted /dev/nvme0n1
(parted) resizepart 6 200GB
```

Create new swap and root

```sh
fdisk
```

Create file systems

```sh
mkfs.ext4 /dev/nvme0n1p8 # root
mkswap /dev/nvme0n1p7
```


### Mount created partition

```sh
mount /dev/nvme0n1p8 /mnt
swapon /dev/nvme0n1p7
mkdir /mnt/efi
mount /dev/nvme0n1p1 /mnt/efi
```

### Chroot

Edit mirrorlist on live system, will later be copied to new system.

```sh
reflector --latest 20 --protocol http --protocol https --sort rate --save /etc/pacman.d/mirrorlist
```

```sh
pacstrap /mnt base linux linux-firmware vim man-db man-pages texinfo dhclient
```

```sh
genfstab -U /mnt >> /mnt/etc/fstab
```

```sh
arch-chroot /mnt
```

### Set time

```sh
ln -sf /usr/share/zoneinfo/Asia/Almaty /etc/localtime
hwclock --systohc
```

### Generate locales

```sh
vim /etc/locale.gen # uncomment en_US.UTF-8 UTF-8 and other locales
locale-gen

vim /etc/locale.conf # LANG=en_US.UTF-8
```


### Networking

```sh
vim /etc/hostname # myhostname
vim /etc/hosts
```

```hosts
127.0.0.1 localhost
::1       localhost
127.0.1.1 myhostname.localdomain myhostname
```


### Root

```sh
passwd
```

### Boot Loader

```sh
pacman -S grub efibootmgr
grub-install --target=x86_64-efi --efi-directory=/efi --bootloader-id=arch
```

```sh
vim /etc/default/grub
vim /etc/grub.d/
```

```sh
pacman -S os-prober
mount /dev/nvme0n1p6 /mnt
grub-mkconfig -o /boot/grub/grub.cfg
vim /etc/default/grub # Add/uncomment GRUB_DISABLE_OS_PROBER=false
grub-mkconfig -o /boot/grub/grub.cfg
```

### Obnoxious beep

```sh
vim /etc/modprobe.d/nobeep.conf # blacklist pcspkr
# rmmod pcspkr # modprobe pcspkr
```

## Post-installation

### Initial network connection

Network: tether

```sh
dhclient
```

### Users

```sh
useradd -m galiguzhinov
passwd galiguzhinov
pacman -S sudo
vim /etc/sudoers # uncomment wheel line
usermod -aG wheel galiguzhinov
```

### Microcode

```sh
pacman -Sy intel-ucode
grub-mkconfig -o /boot/grub/grub.cfg
```

### Num Lock


```sh
cd Downloads/aur
sudo pacman -F fakeroot # find which package has it - it's base-devel
pacman -Sy base-devel
git clone https://aur.archlinux.org/mkinitcpio-numlock.git
cd mkinit*
less PKGBUILD
makepkg -sirc
pacman -U mkinitcpio-numlock-1.0.3.1-any.pkg.tar.zst
```

```sh
vim /etc/mkinitcpio.conf # Add to HOOKS array: numlock before encrypt if any
mkinitcpio -P
```

### Firewall

```sh
pacman -Sy ufw
systemctl start ufw
systemctl enable ufw
```

### GUI

#### Xorg

```sh
pacman -Sy xf86-video-intel
pacman -Sy xorg-server
```

#### i3

```sh
pacman -Sy ttf-opensans
pacman -Sy i3 # all but gaps as it conflicts
pacman -Sy dmenu
mkdir -p ~/.config/i3
cp /etc/i3/config ~/.config/i3/
```

#### Display manager

```sh
pacman -Sy lightdm lightdm-gtk-greeter
systemctl enable lightdm
```

#### Term Emulator

install terminator
