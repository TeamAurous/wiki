# Post Install

This goes over some post installation stuff that you may find useful in order to optimize Arch and improve performance.

## Removing Bloat

If you used [`archinstall`](https://wiki.archlinux.org/title/archinstall) to install Arch, you may have some bloat leftover that came from installing [`Plasma`](https://wiki.archlinux.org/title/KDE).
To remove these unused packages, uninstall `discover`, `oxygen`, and `plasma-vault`.
```bash
$ sudo pacman -Rnsc discover oxygen plasma-vault
```

## Supporting Packages

The following packages provide fixes for some annoying bugs that you may encounter:
- `libappindicator-gtk3` - Fixes blurry icons in electron apps
- `appmenu-gtk-module` - Fixes for GTK menus
- `packagekit-qt5` - Fixes KDE issues

```bash
$ sudo pacman -S libappindicator-gtk3 appmenu-gtk-module packagekit-qt5
```

## Bluetooth

The following commands enable Bluetooth:
```bash
$ sudo sed -i 's|#AutoEnable=false|AutoEnable=true|g' /etc/bluetooth/main.conf
$ sudo systemctl enable --now bluetooth.service
```

## Laptops

[`TLP`](https://wiki.archlinux.org/title/TLP) saves laptop battery power, and optimizes for performance.

Install and enable it by installing the [`tlp`](https://archlinux.org/packages/extra/any/tlp/) package and enabling the service:
```bash
$ sudo pacman -S tlp
$ sudo systemctl enable tlp.service --now
```

## Taming Journal Size

### Limit the file archive size

Limitting the file archive size makes it not go batshit insane apparently:
```bash
$ sudo journalctl --vacuum-size=100M
```

### Limit Time

Limit the time limit for journal files:
```bash
$ sudo journalctl --vacuum-time=2weeks
```

### Applying Changes

Apply the changes by restarting the `systemd-journald` service:
```bash
$ sudo systemctl restart systemd-journald.service
```

## Cleaning

### Packages

Keep only one version after every action:
```bash
$ yay -S pacman-contrib
$ sudo mkdir /etc/pacman.d/hooks
$ curl -LR https://pastebin.com/raw/WCm35wZC > /etc/pacman.d/hooks/clean_package_cache.hook
```

### Disabling Core Dumps

Disabling core dumps can be done by editing `/etc/systemd/coredump.conf` and changing/adding the following:

``` /etc/systemd/coredump.conf
[Coredump]
Storage=none
```

Apply changes:
```bash
$ sudo systemctl daemon-reload
```

## Performance

### Gamemode

[`gamemode`](https://github.com/FeralInteractive/gamemode) is a package where games can request a set of optimizations to be temporarily applied.

#### Installation

Install `gamemode` by download it's package and enabling its service:
```bash
$ sudo pacman -S gamemode
$ systemctl --user enable gamemoded && systemctl --user start gamemoded
```

#### Usage

Some games automatically integrate GameMode support (listed in its GitHub).

For games that do not support integrated GameMode support:
```bash
$ gamemoderun <path to game>
```

### Earlyoom

[`earlyoom`](https://github.com/rfjakob/earlyoom) is a memory manager for Linux. It frees up and manages RAM, and prevents OS freezing:
```bash
$ sudo pacman -S earlyoom
$ sudo systemctl enable --now earlyoom
```

You can also decrease RAM swappiness with the following command:
```bash
$ sudo sysctl -w vm.swappiness = 10
```

### Improving I/O Performance

You can use the BFQ schedule to [improve I/O performance](https://wiki.archlinux.org/title/Improving_performance#Changing_I/O_scheduler).

Create the `60-ioschedulers.rules` file and place it in `/etc/udev/rules.d/` and add the following based on your configuration:

``` /etc/udev/rules.d/60-ioschedulers.rules
# HDD
ACTION=="add|change", KERNEL=="sd[a-z]*", ATTR{queue/rotational}=="1", ATTR{queue/scheduler}="bfq"

# SSD
ACTION=="add|change", KERNEL=="sd[a-z]*|mmcblk[0-9]*", ATTR{queue/rotational}=="0", ATTR{queue/scheduler}="bfq"

# NVMe SSD
ACTION=="add|change", KERNEL=="nvme[0-9]*", ATTR{queue/rotational}=="0", ATTR{queue/scheduler}="none"
```

### Pacman Tweaks

Setup mirrors and parallel downloading for [`Pacman`](https://wiki.archlinux.org/title/pacman) to improve performance:
```bash
$ pacman -S pacman-contrib reflector rsync
$ iso=$(curl -4 ifconfig.co/country-iso)
$ cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.bak
$ sed -i 's/^#Para/Para/' /etc/pacman.conf
```

### Make

Use all cores for faster make:
```bash
$ nc=$(grep -c ^processor /proc/cpuinfo)
$ sudo sed -i 's/#MAKEFLAGS="-j2"/MAKEFLAGS="-j$nc"/g' /etc/makepkg.conf
$ sudo sed -i 's/COMPRESSXZ=(xz -c -z -)/COMPRESSXZ=(xz -c -T $nc -z -)/g' /etc/makepkg.conf
```

### VS Code

Fix for "too many files" in VS Code:
```bash
$ echo fs.inotify.max_user_watches=524288 | sudo tee /etc/sysctl.d/40-max-user-watches.conf && sudo sysctl --system
```

### Network Optimizations

All of the following `sysctl` commands improve network performance. Use at your own risk:
```bash
sudo sysctl -w net.core.netdev_max_backlog = 16384
sudo sysctl -w net.core.somaxconn = 8192
sudo sysctl -w net.core.rmem_default = 1048576
sudo sysctl -w net.core.rmem_max = 16777216
sudo sysctl -w net.core.wmem_default = 1048576
sudo sysctl -w net.core.wmem_max = 16777216
sudo sysctl -w net.core.optmem_max = 65536
sudo sysctl -w net.ipv4.tcp_rmem = 4096 1048576 2097152
sudo sysctl -w net.ipv4.tcp_wmem = 4096 65536 16777216
sudo sysctl -w net.ipv4.udp_rmem_min = 8192
sudo sysctl -w net.ipv4.udp_wmem_min = 8192
sudo sysctl -w net.ipv4.tcp_fastopen = 3
sudo sysctl -w net.ipv4.tcp_max_syn_backlog = 8192
sudo sysctl -w net.ipv4.tcp_max_tw_buckets = 2000000
```

## Fonts

Install missing fonts by downloading the packages `noto-fonts`, `noto-fonts-cjk`, `ttf-dejavu`, `ttf-liberation`, and `ttf-opensans`:
```bash
$ sudo pacman -S noto-fonts noto-fonts-cjk ttf-dejavu ttf-liberation ttf-opensans
```

Windows fonts can be installed by downloading `tf-ms-win11-auto` from the `AUR`.

If using `yay`, install using:
```bash
$ yay -S tf-ms-win11-auto
```

Google Sans can be found [`here`](https://github.com/sahibjotsaggu/Google-Sans-Fonts/blob/master/GoogleSans-Regular.ttf).

Apple fonts can be downloading from the `AUR` too by installing `otf-san-francisco` and `otf-san-francisco-mono`.

If using `yay`, install using:
```bash
$ yay -S otf-san-francisco otf-san-francisco-mono
```

## Appearance

### Using ZSH
`ZSH` is a customizable shell and in my opinion, better than `bash`.

#### Installation

Install the `ZSH` package and change the default shell:
```bash
$ sudo pacman -S zsh && sudo chsh -s $(which zsh)
```

#### Kali Defaults

If you like how the Kali Linux shell looks like, it can be replicated by doing the following:

1. Ensure `git` and `curl` is installed: `sudo pacman -S git curl`
2. Install [`Oh My Zsh`](https://github.com/ohmyzsh/ohmyzsh) by following the instructions [here](https://github.com/ohmyzsh/ohmyzsh#getting-started).
3. Inside the `/usr/share` directory, clone the following repositories: [`zsh-autosuggestions`](https://github.com/zsh-users/zsh-autosuggestions) and [`zsh-syntax-highlighting`](https://github.com/zsh-users/zsh-syntax-highlighting)
4. Add the [`zshrc`](https://gitlab.com/kalilinux/packages/kali-defaults/-/raw/kali/master/etc/skel/.zshrc) for Kali Linux to your `zshrc` at your own risk.

### Forcing KDE file picker

You can force the KDE file picker in GTK apps by installing `xdg-desktop-portal` and setting an environment variable so programs load KDE-specific APIs.

First, install the `xdg-desktop-portal` package:
```bash
sudo pacman -S xdg-desktop-portal
```

Then, set the appropriate environment variable.

If using `ZSH`, you can add it to your `zshrc` file:
``` ~/.zshrc
export GTK_USE_PORTAL=1
```

If using `bash`, add it to your `bashrc` file:
``` ~/.bashrc
export GTK_USE_PORTAL=1
```

### Firefox/Librewolf UI:

Make the UI for Firefox or Librewolf better by applying a fix by [`black7375`](https://github.com/black7375/Firefox-UI-Fix):

1. Go to/Open `about:profiles`
2. Note/Copy/Save your **Profile Name**
3. Run `bash -c "$(curl -fsSL https://raw.githubusercontent.com/black7375/Firefox-UI-Fix/master/install.sh)"` and choose your theme:
   - Original: Chrome-like UI
   - Photon: Classic Firefox UI
4. Select the profile from the **Profile Name** you noted eariler.
5. After it's finished, go to `about:support` and click on `Clear Startup Cache`

### Colors in Pacman

We all like colors right? You can enable colors in Pacman by just applying the following to its config file:
```bash
$ sudo sed -i 's|#Color|Color|g' /etc/pacman.conf
```

### Enabling Blur in Menus (KDE)

Enable blur in Menus (KDE):

1. Open System Settings > Appearance > Application Style > Widget Style > Applications > Widget Style
2. Ensure Breeze is selected and click "Configure"
3. Click the "Transparency" tab and adjust the slider
4. Go back to settings > Workspace > Desktop behavior > Desktop effects > Blur
5. Click the Configure button. Set Noise strength to 0 and adjust the Blur strength slider
6. Click OK

## Global system adblock

Download and install a global adblock by editing the `hosts` file. It can be done with a one line command too:
```bash
$ sudo curl -LR https://raw.githubusercontent.com/hagezi/dns-blocklists/main/hosts/pro.txt > /etc/hosts
```