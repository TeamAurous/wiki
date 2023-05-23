# Windows VM With GPU Passthrough

There are some cases where using the `WINE` emulator for Linux or `Proton` or other Windows compatibility layers for Linux may not work for the program you are trying to use. In this case, create a Windows virtual machine is your next best option to run these Windows programs, however the performance of a virtual machine can be spotty, and at times, bad, compared to running it on bare metal. The main issue with them is video acceleration. One way to solve this, however, is passing through your entire GPU to your virtual machine, giving you almost bare-metal performance.

This guide will be heavily referenced from the offical Arch Linux [guide](https://wiki.archlinux.org/title/PCI_passthrough_via_OVMF), thus if you need more information or troubleshooting, check out the guide.

## Prerequisites

- A CPU that supports hardware virtualization and IOMMU.
- A motherboard that supports IOMMU
- 2 GPU'S
  - Host GPU will be the one outputting Linux
  - Guest GPU must support UEFI
- Two or more monitors
  - One or more monitors for your host GPU
  - One or more monitors for your guest GPU
- A spare keyboard and mouse is recommended to passthrough to the VM

!!!
It is possible to do this with one GPU (it has to be dedicated) by unloading the video driver before the virtual machine starts, and loading the `VFIO` driver, and doing some more steps. Searching around could lead you to a guide on how to do this.
!!!

!!!
You could exclude the monitor requirement for the host GPU if doing this purely via the command line with SSH, which will not be covered here. The same could said with keyboard and mouse.
!!!

