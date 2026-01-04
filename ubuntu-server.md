# Ubuntu Server installation

I installed [Ubuntu Server 24.04](https://ubuntu.com/download/server).
- onto a 2.5" ssd installed in the front cage
- using the VGA outout
- keyboard connected
- no network connection
- no serial connection
- removed the eUSB module

Because network and serial connections where not configured while the installation, it had to be configured in the OS to get rid off the need for the vga adapter.

The console cable is useful for accessing the bios. For OS access I configured SSH.

## enable serial connection

To enable the serial connection the grub configuration has to be changed.

```bash
    4  cat /etc/default/grub
    5  nano /etc/default/grub
    6  sudo nano /etc/default/grub
    # set line:
    # GRUB_CMDLINE_LINUX_DEFAULT="console=tty0 console=ttyS0,115200n8"
    7  sudo update-grub
    8  sudo reboot

```

> TBD: refine section

## configure network

> TBD

## configure SSH

> TBD
