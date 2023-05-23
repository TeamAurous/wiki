# Installing Arch Linux

This will guide you into installing Arch Linux onto your system, and how to set it up. This guide is heavily based on the original [Arch Installation Guide](https://wiki.archlinux.org/title/installation_guide).

This guide will also assume that you have downloaded, flashed, and booted into the [Arch Linux Live USB](https://wiki.archlinux.org/title/USB_flash_installation_medium).

!!!danger
It is recommended you test inside a VM before proceeding on a real machine in order to make sure what you are doing.
!!!

## Verifying you are in UEFI mode <a name='uefi'></a>
If you are booted from the live USB, verify it's in UEFI mode by listing the [`efivars`](https://wiki.archlinux.org/title/Efivars) directory:
```bash
$ ls /sys/firmware/efi/efivars
```

If the directory does not exist, then the USB is not booted in UEFI mode. Make sure to boot it into UEFI mode before proceeding, or continue if you what you are doing, as this guide assumes you are on UEFI.

## Live USB clock <a name='clock'></a>
Use [`timedatectl`](https://man.archlinux.org/man/timedatectl.1) to ensure that the system clock is accurate:
```bash
$ timedatectl
```

## Internet <a name='internet'></a>
You need to set up internet before going forward with this guide. Steps are going to be different depending on whether you are going to be connecting with Ethernet or WiFi. You can follow the in-depth guide from the Arch Wiki [here](https://wiki.archlinux.org/title/Network_configuration), otherwise, here is a summarized/easy-to-follow version:

==- Ethernet <a name='ethernet'></a>

Arch should come pre-installed with Ethernet drivers in the kernel, so connecting your computer to the router should be as simple as plugging in the cable. If you are still unable to connect to the network, follow [this guide](https://wiki.archlinux.org/title/Network_configuration/Ethernet#Device_driver) to check the drivers and load modules.
===

==- WiFi <a name='wifi'></a>
WiFi is a bit more complicated. You have to set up drivers, connect to the router, and set a DHCP thing or something.

#### Checking drivers <a name='drivers'></a>
Verify that a kernel driver is loaded for the WiFi card by checking the output of `lspci -k` for a PCI(e) network card or `lsusb -v` for a USB card. It should say that a kernel driver is in use, for example:
```bash $ lspci -k
...
06:00.0 Network controller: Intel Corporation WiFi Link 5100
		Subsystem: Intel Corporation WiFi Link 5100 AGN
		Kernel driver in use: iwlwifi
		Kernel modules: iwlwifi
...
```

> Tip: If the output is way too long that it goes off screen, try piping it to less in order to go line by line: `lspci -k | less`, press `q` to quit.

If there is no driver loaded, then install the necessary driver by following [this guide](https://wiki.archlinux.org/title/Network_configuration/Wireless#Installing_driver/firmware), or if you have a Broadcom card, continue reading.

##### Broadcom <a name='broadcom'></a>
If using a Broadcom card, you can load the [`broadcom-wl`](https://archlinux.org/packages/?name=broadcom-wl) driver that is already on the live USB. To load it, we must block the conflicting drivers that are trying to grab the card.

To block, assuming you are in UEFI mode, edit the [kernel parameters](https://wiki.archlinux.org/title/kernel_parameters) to add the following:
```
module_blacklist=b43,b43legacy,ssb,bcm43xx,brcm80211,brcmfmac,brcmsmac,bcma
```

You can use [`systemd-boot`](https://wiki.archlinux.org/title/systemd-boot) to accomplish this by pressing `E` to edit the kernel boot parameters before booting into Arch.

#### Connecting to the network. <a name='connecting'></a>
Connecting to the internet can be a hassle in Arch, however assuming that the kernel drivers are loaded correctly, the live USB already comes with tools that make connecting to the internet easy. One such tool, which we are going to use, is called [`iwd`](https://wiki.archlinux.org/title/iwd).

To start, launch the `iwd` shell:
```bash
$ iwctl
```

Next, list your available network adapters:
```bash
[iwd]# device list
```

Note down the the WiFi adapter that you are going to be using as we are going to use it for the following commands.

You can initiate a scan for available networks by doing:
```bash
[iwd]# station <device> scan
```

Then, list all of the available networks it scanned:
```bash
[iwd]# station <device> get-networks
```

Finally, connect to a network:
```bash
[iwd]# station <device> connect <SSID>
```

> If prompted, input the password for the network

To exit `iwd`
```bash
[iwd]# exit
```

===

### Setting up DHCP <a name='dhcp'></a>
Now that we are connected to the network, we gotta setup DHCP to access the internet. This is required for both ethernet and WiFi. We are going to be using a tool called [`dhclient(8)`](https://linux.die.net/man/8/dhclient) in the live USB, and [`dhcpcd`](https://wiki.archlinux.org/title/dhcpcd) on the installation (which we will cover later on) to accomplish this.

Simply, just run:
```bash
$ dhclient -v
```

### Testing the connection <a name='test'></a>
You can test if your internet is set up correctly by pinging a server:
```bash
$ ping 1.1.1.1
```

---

## Choosing Installation Method

Now, choose whether to install Arch Linux manually or using the [`archinstall`](https://wiki.archlinux.org/title/archinstall) tool.

Installing manually is recommended, as it teaches you how the system works and is recommended by the Arch Linux developers. However, you may choose to use the `archinstall` tool if you do not feel comfortable doing so. Proceed at your own risk.

+++ Manual
## Partitioning <a name='partitioning'></a>
Now, lets setup partitions to install Arch Linux on.

### Dual Boot <a name='dualboot'></a>
> Untested

It is recommended to have Windows installed before you continue with installing Arch. While booted into Windows, open `Disk Management`, and resize your main data partition to make space for Arch, and optionally, a swap partition. Afterwards, continue to [Formatting](#formatting).

### Using the Entire Disk <a name='entiredisk'></a>
First, list your disks by using `fdisk -l | grep "Disk /dev/"`. It should output all the disks that are connecting to the system, for example:
```bash $ fdisk -l | grep  "Disk /dev/"
Partition 1 does not start on physical sector boundary.  
Partition 2 does not start on physical sector boundary.  
Disk /dev/nvme0n1: 465.76 GiB, 500107862016 bytes, 976773168 sectors  
Disk /dev/sda: 476.94 GiB, 512110190592 bytes, 1000215216 sectors  
Disk /dev/sdb: 465.76 GiB, 500107862016 bytes, 976773168 sectors  
Disk /dev/sdc: 1.82 TiB, 2000398934016 bytes, 3907029168 sectors
```

> To view more information about a disk, including its partitions, run `fdisk -l /dev/<disk to examine>`

#### Wiping and partitioning <a name='wiping'>

We will be using a tool called [`parted`](https://wiki.archlinux.org/title/Parted)  to partition our disks.

To begin, start the parted shell:
```bash
$ parted /dev/<disk to partition>
```

Next, format the disk and convert it to a gpt disk by running:
```bash
(parted) mklabel gpt
```

!!!danger
This command erases **everything** on the disk. You have been warned.
!!!

Now, make an EFI partition starting at `1MiB` and ending at `300MiB`
```bash
(parted) mkpart "EFI" fat32 1MiB 300MiB
(parted) set 1 esp on 
```

It is recommended to create a swap partiton. If you do choose to create one, create a swap partition starting at `300MiB` and end it at whatever size you choose how big your swap partition is:
```bash
(parted) mkpart "Swap" linux-swap 300MiB 16GiB
```

Next, fill the rest of the remaining disk space starting from wherever you ended your swap partition:
```bash
(parted) mkpart "Arch" ext4 16GiB 100%
```

Lastly, quit `parted`:
```bash
(parted) quit
```

Verify that the partitions were created successfully:
```bash
$ fdisk -l /dev/<disk that was partitioned>
```

#### Formatting <a name='formatting'></a>

Format the EFI partition **if you are not dual-booting**:
```bash
$ mkfs.fat -F32 /dev/<EFI partition>
```

Format the swap partition:
```bash
$ mkswap /dev/<swap partition>
$ swapon /dev/<swap parition>
```

Format the root partition
```bash
$ mkfs.ext4 /dev/<root partition>
```

Finally, mount the necessary partitions so we can work with them:
```bash
$ mount /dev/<root partition> /mnt
$ mount /dev/<efi partition> /mnt/efi
```
!!!
You may need to create the `efi` directory if it fails by running `mkdir /mnt/efi`
!!!

## Installing Arch <a name='installing'></a>

Now we have to install Arch onto the mounted root partition. We are going to use a tool called [`pacstrap`](https://wiki.archlinux.org/title/Pacstrap).

Install the [`base`](https://archlinux.org/packages/core/any/base/), [`linux`](https://archlinux.org/packages/core/x86_64/linux/), and [`linux-firmware`](https://archlinux.org/packages/core/any/linux-firmware/) packages to the root partition:
```bash
$ pacstrap /mnt base linux linux-firmware
```

After that is done you may want to install some other basic tools before chroot'ing to the install. This can include a text editor or other utilities as the [`base`](https://archlinux.org/packages/core/any/base/) package doesn't provide all utilities. You can install other tools later with the `pacman` command, but for now, install the essential stuff like a text editor:
```bash
$ pacstrap /mnt vim
```

> You may want to install [`emacs`](https://wiki.archlinux.org/title/emacs) and [`vi`](https://wiki.archlinux.org/title/Vi) alongside [`vim`](https://wiki.archlinux.org/title/vim) as other applications like [`sudo`](https://wiki.archlinux.org/title/sudo) might look for them instead of `vim` or another text editor you install

Next, generate the [`fstab`](https://wiki.archlinux.org/title/fstab) file for the system:
```bash
$ genfstab -U /mnt >> /mnt/etc/fstab
```

Check the resulting file in case of any error:
```bash
$ cat /mnt/etc/fstab
```

## Setting up Arch <a name='settingup'></a>

Now it's time to set up the Arch system. Change root to the root partition by running
```bash
$ arch-chroot /mnt
```

This will essentially locally "ssh" into the system, This is a very useful tool.

### Setting up time <a name='settingtime'></a>

It is recommened to use `UTC` time if you are dual-booting multiple OS's as it causes less issues than using either exclusively `localtime` or mix and matching.

#### Changing standard on Linux <a name='stdlinuxtime'></a>

To change the hardware clock time standard to `localtime`, use:
```bash
$ timedatectl set-local-rtc 1
```

To revert:
```bash
$ timedatectl set-local-rtc 0
```

#### Changing Windows to use UTC instead of localtime <a name='winutc'></a>

If you decide to be sane and use `UTC` on Linux, then you must do the same on Windows. 

Open up an administrator Command Prompt on Windows and use:
```batch
reg add "HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\TimeZoneInformation" /v RealTimeIsUniversal /d 1 /t REG_DWORD /f
```

> I would do this after you are finished with the live USB, before booting into Arch for the first time.

#### Changing Time Zone on Arch <a name='timezone'></a>

To check the current time zone:
```bash
$ timedatectl status
```

To list available zones:
```bash
$ timedatectl list-timezones
```

To set the timezone:
```bash
$ timedatectl set-timezone Zone/SubZone
```

Example using Central US Chicago time:
```bash
$ timedatectl set-timezone America/Chicago
```

Then synchronise the hardware clock:
```bash
$ hwclock --systohc
```

### Setting up locale's <a name='settinglocale'></a>
**Don't mess this up.** Trust me it's gonna save a lot of headaches later.

Generate the locale:
```bash
$ locale-gen
```

Now set the `LANG` variable:
```bash
$ echo "LANG=en_US.UTF-8" >> /etc/locale.conf
```

Now edit the `/etc/locale.gen` file and uncomment the following lines (most likely on line 13 and 14, if not, just look for them):
``` /etc/locale.gen
 13| en_US ISO-8859-1
 14| en_US.UTF-8 UTF-8
```

Finally, regenerate the locale by running
```bash
$ locale-gen
```

### Setting up hosts <a name='settinghost'></a>

Set up the [`hostname`:](https://wiki.archlinux.org/title/Hostname)
```bash
$ echo "<hostname>" > /etc/hostname
```

The hostname is basically the PC name, and is used to uniquely identify the device on the network. See [RFC-1178](https://datatracker.ietf.org/doc/html/rfc1178) for choosing a hostname. Examples could be `arch-pc`, `arch`, `gaming-pc`, etc.

Now setup the [`hosts(5)`](https://man.archlinux.org/man/hosts.5.en) file:
```bash
$ echo -e "127.0.0.1\tlocalhost\n::1\t\tlocalhost" >> /etc/hosts
```

### Finishing Installation <a name='settingdone'></a>

Change the root password by using [`passwd(1)`](https://man.archlinux.org/man/passwd.1.en):
```bash
$ passwd
```

Now, install the following packages to aid us in the post-install section. Of course, you can choose whether or not you want to install any of these but these are highly recommended to make our lives easier:

- [`sudo`](https://wiki.archlinux.org/title/sudo) - Ability to run commands as sudo
- [`dhcpcd`](https://wiki.archlinux.org/title/dhcpcd) - A DHCP and DHCPv6 client
- [`iwd`](https://wiki.archlinux.org/title/iwd) - Tool we used earlier for WiFi
- Any other packages you may need, such as WiFi drivers, etc

```bash
$ pacman -S sudo dhcpcd iwd
```

> If once you have booted the Arch install and you might have some packages missing, you can boot back into the Arch live USB and chroot back into the system. Just remount the root partition as we did in the [partitioning](#formatting) section and run the `arch-chroot` command like we did in the [setting up](#settingup) section.

## Installing GRUB <a name='grub'></a>
Finally, before we boot, we have to install a [boot loader](https://wiki.archlinux.org/title/Arch_boot_process#Boot_loader) and install CPU [microcode](https://wiki.archlinux.org/title/Microcode):

```bash
$ pacman -S grub efibootmgr
```

Now install the microcode, if you have a AMD CPU, install [`amd-ucode`](https://archlinux.org/packages/core/any/amd-ucode/), if you have an Intel CPU, install [`intel-ucode`](https://archlinux.org/packages/core/any/intel-ucode/):
```bash
$ pacman -S <microcode package>
```

Now install the [`GRUB`](https://wiki.archlinux.org/title/GRUB) bootloader to the EFI drive:
```bash
$ grub-install --target=x86_64-efi --efi-directory=/efi --bootloader-id=GRUB
```

Then, create the configuration file:
```bash
$ grub-mkconfig -o /boot/grub/grub.cfg
```

You can make any changes by editing the `GRUB` configuration file located in `/etc/default/grub`, just be sure to rerun the `grub-config` command again just like above.

Finally, you can exit out of the shell by just typing in `exit`, and it should put you right back into the live USB shell.
```bash
$ exit
```

## Booting Arch <a name='booting'></a>
Before shutting down the live USB, make sure to unmount the drives in case of any error

Run in the following order:
```bash
$ umount -R /mnt/efi
$ umount -R /mnt
```

Check for any busy errors after each command. If all goes well, you can finally shutdown the system and boot into Arch!

> To restart, `shutdown -r now`

## Finalization <a name='postinstall'></a>

### Network <a name='postnetwork'></a>

Connect to the network the same as before in the [Internet](#internet) section.

Instead of following the same instructions as in the [Setting up DHCP](#dhcp) section, we are gonna use [`dhcdcp`](https://wiki.archlinux.org/title/dhcpcd) do setup `DHCP`.

After installing, it's as easy as [starting and enabling](https://wiki.archlinux.org/title/Start/enable) the `dhcpcd.service` daemon:
```bash
$ systemctl start dhcpcd.service
$ systemctl enable dhcpcd.service
```

### Setting up users <a name='users'></a>

Being logged in as root all the time is **dangerous** and **not recommended**, therefore, we must create a new user without root privaleges.

Create a new user:
```bash
$ useradd -m <username>
```

And now set it's password:
```bash
$ passwd <username>
```

### Setting up sudo <a name='sudo'></a>

Add the newly created user to the `wheel` [group](https://wiki.archlinux.org/title/users_and_groups):
```bash
$ usermod -aG wheel <username>
```

Then, edit the `sudoers` file and uncomment this line (most likely line 84):
``` /etc/sudoers
 84| %wheel ALL=(ALL) ALL
```

> You may have to use a application called `visudo` which is installed alongside `sudo` to edit the `sudoers` file. This app requires `vi` to be installed.

To test out `sudo`, log out of the `root` account and login to your user. Then, run the [`whomai(1)`](https://man.archlinux.org/man/whoami.1.en) as root to verify if `sudo` works: 
```bash
$ sudo whoami
```

If the output says `root` then it's set up.

### Adding the multilib Repository <a name='pacman'></a>

The [`multilib`](https://wiki.archlinux.org/title/official_repositories#multilib) repository contains 32-bit libraries and applications that can be used on 64-bit installs.

Add the `multilib` repository to Arch by editting and uncommenting the `[multilib]` section:
``` /etc/pacman.conf
[multilib]
Include = /etc/pacman.d/mirrorlist
```

Then, [upgrade (`sudo pacman -Syu`)](https://wiki.archlinux.org/title/Upgrade) the system.

### Installing Yay Package Manager <a name='yay'></a>

The [`yay`](https://github.com/Jguer/yay) package manager allows us to install packages from the [`AUR`](https://aur.archlinux.org/). The `AUR` is a huge community-driven repository that contains user-submitted packages.

Install the dependencies [`base-devel`](https://archlinux.org/packages/core/any/base-devel/) and [`git`](https://archlinux.org/packages/extra/x86_64/git/):
```bash
$ sudo pacman -S base-devel git
```

Then, clone the repository from the `AUR`:
```bash
$ git clone https://aur.archlinux.org/yay.git
```

`cd` into the repository and finally, run [`mkpkg`](https://wiki.archlinux.org/title/makepkg):
```bash
$ makepkg -si
```

### Setting up Audio <a name='audio'></a>

By default, Arch does not have audio configured. To setup audio, install [`pulseaudio`](https://wiki.archlinux.org/title/PulseAudio) and [`alsa-utils`](https://github.com/alsa-project/alsa-utils):
```bash
$ pacman -Sy alsa-utils pulseaudio
```

For more information on setting up audio, see this [page.](https://wiki.archlinux.org/title/sound_system)

### Setting up video drivers

Use the following table to determine which video driver to install:

|Brand|Driver|Documentation|
|-----|------|-------------|
|AMD|[`mesa`](https://archlinux.org/packages/extra/x86_64/mesa/), [`vulkan-radeon`](https://archlinux.org/packages/extra/x86_64/vulkan-radeon/)|[`AMDGPU`](https://wiki.archlinux.org/title/AMDGPU)|
|Intel|[`mesa`](https://archlinux.org/packages/extra/x86_64/mesa/)|[Intel graphics](https://wiki.archlinux.org/title/intel_graphics)|
|NVIDIA|[`nvidia`](https://archlinux.org/packages/extra/x86_64/nvidia/)|[NVIDIA](https://wiki.archlinux.org/title/NVIDIA)

### Setting up KDE

[KDE](https://wiki.archlinux.org/title/KDE) Plasma is a desktop environment (DE) that is customizable and cool.

To install base `KDE` without much bloat, install the [`xorg-xinit`](https://wiki.archlinux.org/title/xinit) and [`plasma-desktop`](https://archlinux.org/packages/extra/x86_64/plasma-desktop/) packages:
```bash
$ sudo pacman -S xorg-xinit plasma-desktop
```

[Create](https://wiki.archlinux.org/title/Help:Reading#Append,_add,_create,_edit) the `.xinitrc` file in `~/.xinitrc` and add the following lines:
```bash ~/.xinitrc
export DESKTOP_SESSION=plasma
exec startplasma-x11
```

Finally, install any browsers, text editors, file managers, terminal emulators, etc. that you may need. Some examples include [`dolphin`](https://wiki.archlinux.org/title/Dolphin), [`firefox`](https://wiki.archlinux.org/title/firefox), [`konsole`](https://wiki.archlinux.org/title/Konsole), and [`kate`](https://archlinux.org/packages/extra/x86_64/kate/).

To start the DE, in the terminal, use:
```bash
$ startx
```

and if everything is setup correctly, Plasma should start and ready to use!
+++ archinstall
## Run the Installer <a name='run'></a>

Before we run the installer, we must install `python` and the actual installer, `archinstall`, itself:
```bash
$ pacman -S python archinstall
```

- Step 2: Run archinstall
  - Setup:
    `pacman -Syy python archinstall`
    `archinstall`
  1. Mirror region > choose a region
  2. Drive(s) > choose your disk
  3. Setting up partitions
    - Installing alongside Windows:
      1. In Windows make sure you created a new empty partiton. Use diskmgmt
      2. Disk layout > "Select what to do"
      3. Choose "Mark/Unmark a parition to be be formatted" and choose your empty linux partition
      4. Choose "Set a desired filesystem for a partition", and use ext4
      5. Choose "Assign mount-point for a partition" > select your ext4 partition > type `/`
      6. Choose "Assign mount-point for a partition" > select parition 0 (boot is True) > type `/boot`
      7. Save and exit
    - Wiping the entire drive:
      - Disk layout > "Wipe all selected..." > ext4 > no (no separate partition for /home)
  4. Select bootloader > systemd-boot (WARNING: you MUST select systemd-boot if you installing alongside Windows)
  5. User account > Add a user > username > userpassword > confirm pass > yes (be a superuser) > Confirm and exit
  6. Root password > Set same as userpassword
  7. Profile > desktop > kde > choose driver (use either ATI for amd, Intel, or Nvidia proprietary)
  8. Audio > pulseaudio
  9. Network configuration > Copy ISO.. for ethernet, Use NetworkManager for wifi
  10. Timezone > choose a timezone
  11. Optional repositories > select multilib and press enter
  12. Install and wait

----------------

Post-Install Guide for Arch distros:
  - Remove stupid ass bullshit from plasma: `sudo pacman -Rnsc discover oxygen plasma-vault`
  - Other support packages:
    - Fixes blurry icons in Electron programs: `sudo pacman -S libappindicator-gtk3`
    - Fixes for GTK3 menus: `sudo pacman -S appmenu-gtk-module`
    - Fixes various issues in KDE apps: `sudo pacman -S packagekit-qt5`
  - Bluetooth:
    `sudo sed -i 's|#AutoEnable=false|AutoEnable=true|g' /etc/bluetooth/main.conf`
    `sudo systemctl enable --now bluetooth.service`
  - Laptops:
    - Tlp- battery auto-tuning option for laptops:
      `sudo pacman -S tlp`
      `sudo systemctl enable tlp.service --now`
  - Cleaning:
    - Taming the journal's size (reference: https://wiki.archlinux.org/title/Systemd/Journal#Clean_journal_files_manually):
      1. Limit journal file archive to 100M (makes it not go batshit insane): `sudo journalctl --vacuum-size=100M`
      2. Limit journal files to 2 weeks: `sudo journalctl --vacuum-time=2weeks`
      3. `sudo systemctl restart systemd-journald.service`
    - Auto clean packages- Only keep 1 version after every action
      `yay -S pacman-contrib`
      `sudo mkdir /etc/pacman.d/hooks`
      `curl -LR https://pastebin.com/raw/WCm35wZC > /etc/pacman.d/hooks/clean_package_cache.hook`
    - Disable core dumps:
      1. Edit `sudo gedit /etc/systemd/coredump.conf`
      2. Under `[Coredump]` uncomment `Storage=external `and replace it with `Storage=none`
      3. Then run `sudo systemctl daemon-reload`
  - Performance:
    - Setting up Video Acceleration:
      1. Install ffmpeg: `sudo pacman -S ffmpeg`
      2. Find the VA-API package corresponding to the drivers you chose in archinstall: https://wiki.archlinux.org/title/Hardware_video_acceleration#Installation
        - Intel (2014 and newer): `sudo pacman -S libva-media-driver`
        - Nvidia proprietary: `sudo pacman -S nvidia-utils && yay -S nvidia-vaapi-driver`
          - Also edit kernel mode setting: https://github.com/korvahannu/arch-nvidia-drivers-installation-guide#step-3-enabling-drm-kernel-mode-setting
        - AMD/ATI: `sudo pacman -S libva-mesa-driver`
      3. Install libva-utils: `sudo pacman -S libva-utils libva-vdpau-driver`
      4. After doing this, look at the output of `vainfo` to make sure everything is fine. Lines ending in `VAEntrypointVLD` or `VAEntrypointEncSlice` indicate success
      - Firefox:
        - Go to about:config page and set media.ffmpeg.vaapi.enabled to true
        - Check the VA-API acceleration state at about:support page, look at HARDWARE_VIDEO_DECODING row. If there's available by default, you're running on hardware by default
    - Microcode updates (reference: https://wiki.archlinux.org/index.php/Microcode):
      - AMD: `sudo pacman -S amd-ucode`
        - Edit `/boot/loader/entries/arch.conf` so that the first initrd line is the following: `initrd  /amd-ucode.img`
      - Intel: `sudo pacman -S intel-ucode`
        - Edit `/boot/loader/entries/arch.conf` so that the first initrd line is the following: `initrd  /intel-ucode.img`
      - If you are using Grub bootloader:
          `sudo grub-mkconfig -o /boot/grub/grub.cfg`
    - Gamemode- On demand performance optimization: https://github.com/FeralInteractive/gamemode
      `sudo pacman -S gamemode`
      `systemctl --user enable gamemoded && systemctl --user start gamemoded`
    - Earlyoom- Automatically free and manage RAM, and prevent OS freezing:
      `sudo pacman -S earlyoom`
      `sudo systemctl enable --now earlyoom`
    - Using the BFQ schedule to improving I/O performance (reference: https://wiki.archlinux.org/index.php/Improving_performance#Changing_I/O_scheduler):
      - Create this file and fill it in as below `sudo gedit /etc/udev/rules.d/60-ioschedulers.rules`
        - For NVMe: `ACTION=="add|change", KERNEL=="nvme[0-9]*", ATTR{queue/scheduler}="none"`
        - For SSD and eMMC: `ACTION=="add|change", KERNEL=="sd[a-z]|mmcblk[0-9]*", ATTR{queue/rotational}=="0", ATTR{queue/scheduler}="bfq"`
        - For HDD rotating disks: `ACTION=="add|change", KERNEL=="sd[a-z]", ATTR{queue/rotational}=="1", ATTR{queue/scheduler}="bfq"`
    - Pacman tweaks:
      - Set up mirrors for optimal download:
        `pacman -S pacman-contrib reflector rsync`
        `iso=$(curl -4 ifconfig.co/country-iso)`
        `cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.bak`
	  - Parallel downloading:
	    `sed -i 's/^#Para/Para/' /etc/pacman.conf`
    - Use all cores for make:
      `nc=$(grep -c ^processor /proc/cpuinfo)`
      `sudo sed -i 's/#MAKEFLAGS="-j2"/MAKEFLAGS="-j$nc"/g' /etc/makepkg.conf`
      `sudo sed -i 's/COMPRESSXZ=(xz -c -z -)/COMPRESSXZ=(xz -c -T $nc -z -)/g' /etc/makepkg.conf`
    - Fix "too many files" error in Visual Studio Code:
      `echo fs.inotify.max_user_watches=524288 | sudo tee /etc/sysctl.d/40-max-user-watches.conf && sudo sysctl --system`
    - Sysctl network optimizations:
      `sudo sysctl -w net.core.netdev_max_backlog = 16384`
      `sudo sysctl -w net.core.somaxconn = 8192`
      `sudo sysctl -w net.core.rmem_default = 1048576`
      `sudo sysctl -w net.core.rmem_max = 16777216`
      `sudo sysctl -w net.core.wmem_default = 1048576`
      `sudo sysctl -w net.core.wmem_max = 16777216`
      `sudo sysctl -w net.core.optmem_max = 65536`
      `sudo sysctl -w net.ipv4.tcp_rmem = 4096 1048576 2097152`
      `sudo sysctl -w net.ipv4.tcp_wmem = 4096 65536 16777216`
      `sudo sysctl -w net.ipv4.udp_rmem_min = 8192`
      `sudo sysctl -w net.ipv4.udp_wmem_min = 8192`
      `sudo sysctl -w net.ipv4.tcp_fastopen = 3`
      `sudo sysctl -w net.ipv4.tcp_max_syn_backlog = 8192`
      `sudo sysctl -w net.ipv4.tcp_max_tw_buckets = 2000000`
    - Decrease RAM swappiness: `sudo sysctl -w vm.swappiness = 10`
  - Fonts:
    - Use clearer font typing: https://github.com/pdeljanov/infinality-remix/issues/13#issuecomment-1263878169
    - Estimate screen DPI: https://sven.de/dpi
    - Install missing fonts: `sudo pacman -S noto-fonts noto-fonts-cjk ttf-dejavu ttf-liberation ttf-opensans`
  - Windows fonts: `yay -S tf-ms-win11-auto`
    - Other font downlods:
      - Google Sans: https://github.com/sahibjotsaggu/Google-Sans-Fonts/blob/master/GoogleSans-Regular.ttf
      - SF Pro Font: `yay -S otf-san-francisco otf-san-francisco-mono`
    - Nerd mono fonts: https://www.nerdfonts.com/font-downloads
  - Force the KDE file picker in GTK apps:
    - Handler for file picker portals: `sudo pacman -S xdg-desktop-portal`
    - Set env variable so programs load KDE-specific APIs `export GTK_USE_PORTAL=1`
  - Appearance:
    - Zsh- Configurable shell: `sudo pacman -S zsh && sudo chsh -s $(which zsh)`
      - Kali defaults:
        `cd /usr/share`
        `git clone https://github.com/zsh-users/zsh-autosuggestions`
        `git clone https://github.com/zsh-users/zsh-syntax-highlighting`
        `curl -LR https://gitlab.com/kalilinux/packages/kali-defaults/-/raw/kali/master/etc/skel/.zshrc > ~/.zshrc`
      - Oh My Zsh- Zsh theme engine: https://github.com/ohmyzsh/ohmyzsh#getting-started
    - Librewolf/Firefox UI Fix:
      - Open `about:support` in Firefox
        - Note your `Profile Folder` name
        - Run `bash -c "$(curl -fsSL https://raw.githubusercontent.com/black7375/Firefox-UI-Fix/master/install.sh)"` and choose your theme
          - Original: Chrome-like UI
          - Photon: Classic Firefox UI
        - Select the profile from the `Profile Folder` name from earlier
        - Once it's finished, go back to the `about:support` page in Firefox and click "Clear startup cache..."
    - Enable colors in pacman:
      `sudo sed -i 's|#Color|Color|g' /etc/pacman.conf`
    - Enable blur in Menus (KDE):
      1. Open System Settings > Appearance > Application Style > Widget Style > Applications > Widget Style
      2. Ensure Breeze is selected and click "Configure"
      3. Click the "Transparency" tab and adjust the slider
      4. Go back to settings > Workspace > Desktop behavior > Desktop effects > Blur
      5. Click the Configure button. Set Noise strength to 0 and adjust the Blur strength slider
      6. Click OK
  - Global system adblock:
    `sudo curl -LR https://raw.githubusercontent.com/hagezi/dns-blocklists/main/hosts/pro.txt > /etc/hosts`

----------------

Useful Programs

*Note: Package names will be listed `like this`. Use `sudo pacman -S <pkgname>`*

System:
  - Pamac- Manjaro's Arch software center: `pamac-gtk`
  - Nvim- Text editor `neovim`
    - Astrovim- Plugin pack: https://astronvim.com/
  - System Monitor- KDE task manager: `plasma-systemmonitor`
  - Latte Dock: Cleaner Dock: `latte-dock` https://github.com/KDE/latte-dock
  - UFW- Linux firewall: https://wiki.archlinux.org/title/Uncomplicated_Firewall
    `sudo pacman -S ufw gufw`
    `sudo systemctl enable --now ufw`
  - Pavucontrol- Volume control: `pavucontrol` https://freedesktop.org/software/pulseaudio/pavucontrol/
Tools:
  - Kate- Text editor: `kate` https://kate-editor.org/
  - Qalculate- Calculator: `qalculate-gtk` https://qalculate.github.io/
  - Flameshot- Screenshot tool: `flameshot` https://flameshot.org/
  - OnlyOffice- Microsoft Office alternative: `yay -S onlyoffice-bin` https://www.onlyoffice.com/
  - Noisetorch- Mic noise supression: `yay -S noisetorch` https://github.com/noisetorch/NoiseTorch
  - FSearch- Everything Search equivalent for linux: `fsearch` https://github.com/cboxdoerfer/fsearch
  - Okular- PDF viewer: `okular` https://okular.kde.org/
  - LibreWolf- Hardened Firefox without telemetry:
      `yay -S librewolf-bin`
      `sudo pacman -Rns firefox  # remove Firefox if nessecary`
      - Scroll up for LibreWolf UI fix
Images/Video:
  - Video editors:
    - Kdenlive: https://kdenlive.org/en/
    - Shotcut: https://www.shotcut.org/
  - KolourPaint- Mspaint altertnative: `kolourpaint` http://www.kolourpaint.org/screenshots.html
  - Gwenview- Image viewer: `gwenview`
  - SimpleScreenRecorder- Easy screen recorder: `simplescreenrecorder` https://github.com/MaartenBaert/ssr
  - MPV- Video player: `mpv` https://rentry.co/MPV-Config
    - Modern UI: https://github.com/cyl0/ModernX
    - Scripts: https://github.com/mpv-player/mpv/wiki/User-Scripts, Notables: https://i.imgur.com/ZU8r5r4.png
  - Yt-dlp Gui: Universal video downloader: `yay -S ytdlp-gui`
Gaming/Emulation:
  - Lutris- Game manager/windows emulator: `lutris` https://lutris.net/
  - Proton- Compatibility for Windows games on Steam: `yay -S proton` https://github.com/ValveSoftware/Proton
  - Heroic Launcher- Launcher for GOG and Epic Games: https://heroicgameslauncher.com/
  - PlayOnLinux- UI wrapper for Wine to run Windows programs: `playonlinux` https://www.playonlinux.com/en/
  - GOverlay- OpenGL gaming overlay, similar to MSI Afterburner's overlay: `goverlay` https://github.com/benjamimgois/goverlay
+++