# Useful Programs

!!! How to install these programs
Packages listed `like this` are the package names. They can be installed with `pacman`.

Packages with <sup>`aur`</sup> come from the `AUR`. They can be installed with `yay` or another AUR package manager.

Packages with <sup>`git`</sup> does not have a package on any repository (afaik). Installation instructions can be found on the GitHub page of the package, which is linked.
!!!

### System:

- [`pamac-aur-git`](https://aur.archlinux.org/packages/pamac-aur-git)<sup>`aur`</sup>: Manjaro's Arch software center
- [`neovim`](https://archlinux.org/packages/extra/x86_64/neovim/): Text editor
  - [`astrovim`](https://astronvim.com): Plugin pack
- [`plasma-systemmonitor`](https://archlinux.org/packages/extra/x86_64/plasma-systemmonitor/): KDE task manager
- [`latte-dock`](https://archlinux.org/packages/extra/x86_64/latte-dock/): Cleaner dock
- [`pavucontrol`](https://archlinux.org/packages/extra/x86_64/pavucontrol/): Volume control/mixer tool
- [`ufw`](https://archlinux.org/packages/extra/any/ufw/) and [`gufw`](https://archlinux.org/packages/extra/any/gufw/): A Linux firewall, activate by doing `sudo systemctl enable --now ufw`

### Tools:

- [`kate`](https://archlinux.org/packages/extra/x86_64/kate/): Text editor
- [`qalculate-gtk`](https://archlinux.org/packages/extra/x86_64/qalculate-gtk/): Calculator
- [`flameshot`](https://archlinux.org/packages/extra/x86_64/flameshot/): Screenshot tool
- [`onlyoffice-bin`](https://aur.archlinux.org/packages/onlyoffice-bin)<sup>`aur`</sup>: Microsoft Office alternatives
- [`noisetorch-git`](https://aur.archlinux.org/packages/noisetorch-git)<sup>`aur`</sup>: Mic noise suppression
- [`fsearch`](https://aur.archlinux.org/packages/fsearch/)<sup>`aur`</sup>: Everything Search equivalent for Linux
- [`okular`](https://archlinux.org/packages/extra/x86_64/okular/): PDF viewer
- [`librewolf-bin`](https://aur.archlinux.org/packages/librewolf-bin): Hardened Firefox without telemetry

### Images/Video:

- [`kdenlive-git`](https://aur.archlinux.org/packages/kdenlive-git)<sup>`aur`</sup> or [`kdenlive-appimage`](https://aur.archlinux.org/packages/kdenlive-appimage)<sup>`aur`</sup>: Video editor
- [`shotcut`](https://archlinux.org/packages/extra/x86_64/shotcut/): Video editor
- [`kolourpaint`](https://archlinux.org/packages/extra/x86_64/kolourpaint/): MSPaint alternative
- [`gwenview`](https://archlinux.org/packages/extra/x86_64/gwenview/): Image viewer
- [`simplescreenrecorder-git`](https://aur.archlinux.org/packages/simplescreenrecorder-git)<sup>`aur`</sup>: Easy screen recorder
- [`mpv`](https://archlinux.org/packages/extra/x86_64/mpv/): Open source and cross platform video player
  - [`ModernX`](https://github.com/cyl0/ModernX)<sup>`git`</sup>: Modern UI
  - [`User Scripts`](https://github.com/mpv-player/mpv/wiki/User-Scripts): Scripts that add functionality to `mpv`
    - Notable Scripts:
      - compressor
      - [autoload](https://github.com/mpv-player/mpv/blob/master/TOOLS/lua/autoload.lua)
      - [chapter-list](https://github.com/CogentRedTester/mpv-scroll-list/blob/master/examples/chapter-list.lua)
      - [contact-sheet](https://github.com/occivink/mpv-gallery-view)
      - gallery-thumbgen
      - [mpv_thumbnail_script_server](https://github.com/marzzzello/mpv_thumbnail_script)
      - [osc_tethys](https://github.com/Zren/mpv-osc-tethys)
      - [playlistmanager](https://github.com/jonniek/mpv-playlistmanager)
      - playlistmanager-save-interactive
      - [playlist-view](https://github.com/occivink/mpv-gallery-view)
      - [quality-menu](https://github.com/christoph-heinrich/mpv-quality-menu)
      - scroll-list
      - [SmartCopyPaste_II](https://github.com/Eisa01/mpv-scripts#smartcopypaste_ii)
      - [sponsorblock](https://github.com/po5/mpv_sponsorblock)
      - [stats](https://github.com/Argon-/mpv-stats/)
      - youtube-search-for-newer-lua
      - [thumbfast](https://github.com/po5/thumbfast)
- [`ytdlp-gui`](https://aur.archlinux.org/packages/ytdlp-gui)<sup>`aur`</sup>: Universal video downloader

### Gaming/Emulation:

- [`lutris`](https://archlinux.org/packages/extra/any/lutris/): Game manager and Windows emulator
- [`proton`](https://aur.archlinux.org/packages/proton)<sup>`aur`</sup>: Compatibility layer for Windows game on Steam
  - **Note:** Valve recommends you use `proton` that is shipped with the Steam client itself, only use this package as a fallback
- [`heroic-games-launcher-bin`](https://aur.archlinux.org/packages/heroic-games-launcher-bin)<sup>`aur`</sup>: Launcher for GOG and Epic Games
- [playonlinux](https://www.playonlinux.com/en/): UI wrapper for Wine to run Windows programs
- [`goverlay-bin`](https://aur.archlinux.org/packages/goverlay-bin)<sup>`aur`</sup>: OpenGL gaming overlay, similar to MSI Afterburner's overlay