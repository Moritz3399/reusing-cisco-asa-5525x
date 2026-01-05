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

Modify the template file `/etc/default/grub`, `update-grub` and `reboot`.

Add or modify the `GRUB_CMDLINE_LINUX_DEFAULT` option to `console=tty0 console=ttyS0,115200n8`.

> The bit rate can be changed from 115200n8 to your needs.

```bash
cat /etc/default/grub
nano /etc/default/grub
sudo nano /etc/default/grub
# set line:
# GRUB_CMDLINE_LINUX_DEFAULT="console=tty0 console=ttyS0,115200n8"
sudo update-grub
sudo reboot

```

## configure network

I did not configure the network while the OS installation.
I followed the [documentation](https://documentation.ubuntu.com/server/explanation/networking/configuring-networks/#dynamic-ip-address-assignment-dhcp-client) to enable dynamic IP addresses for all ports.

```bash
cd /etc/netplan/
sudo nano 99_config.yaml # config see below
sudo chmod 600 99_config.yaml
sudo netplan apply
```

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp3s0:
      dhcp4: true
    enp4s0:
      dhcp4: true
    enp5s0:
      dhcp4: true
    enp6s0:
      dhcp4: true
    enp7s0:
      dhcp4: true
    enp8s0:
      dhcp4: true
    enp9s0:
      dhcp4: true
    enp10s0:
      dhcp4: true
    enp18s0:
      dhcp4: true
```

After connecting a network cable you can use `ip a` to check, if an IP was assigned.

In my case the port labeled `0` was `enp6s0`.

## configure SSH

I installed the OpenSSH server already while OS installation. You can follow [their documentation](https://documentation.ubuntu.com/server/how-to/security/openssh-server/).

I added my public key to the `~/.ssh/authorized_keys` and was good to go.

> After this the console cable is not required anymore.

## noise level

I used a dB measurement app on my phone. In a distance of 15cm on the back of the server the noise level was between 60 and 65 dB. The fans are either no PWM fans or run with max. speed all the time.

While running a CPU stress test with stress-ng there was no change in the noise levels.

The fans are connected with 4pin connectors. There might be a configuration issue, that I need to fix.

I installed `fancontrol` and checked with `pwmconfig`, but there are no pwm modules available.

```bash
sudo apt install fancontrol

sudo pwmconfig
# pwmconfig version 3.6.0
This program will search your sensors for pulse width modulation (pwm)
controls, and test each one to see if it controls a fan on
your motherboard. Note that many motherboards do not have pwm
circuitry installed, even if your sensor chip supports pwm.

We will attempt to briefly stop each fan using the pwm controls.
The program will attempt to restore each fan to full speed
after testing. However, it is ** very important ** that you
physically verify that the fans have been to full speed
after the program has completed.

/usr/sbin/pwmconfig: There are no pwm-capable sensor modules installed
```

## cpu stress test

I used `stress-ng` and `lm-sensors` to run a cpu stress test and check on the noise level and temperatures. 

The cpu is running on 30°C in idle. While a short stress test it went up to 55°C.

```bash
sudo apt install lm-sensors
sudo apt install stress-ng

cd ~
stress-ng -c 20 --timeout 120 # timeout in seconds

# open a second terminal to monitor the cpu temperatures
watch -n 3 sensors
```

## System information
Just some dumps of `screenfetch`,`lscpu`,`lsusb` and `lspci`.

```bash
user@cisco-asa:~$ screenfetch
                          ./+o+-       user@cisco-asa
                  yyyyy- -yyyyyy+      OS: Ubuntu 24.04 noble
               ://+//////-yyyyyyo      Kernel: x86_64 Linux 6.8.0-90-generic
           .++ .:/++++++/-.+sss/`      Uptime: 43m
         .:++o:  /++++++++/:--:/-      Packages: 770
        o:+o+:++.`..```.-/oo+++++/     Shell: bash 5.2.21
       .:+o:+o/.          `+sssoo+/    Disk: 7.5G / 56G (15%)
  .++/+:+oo+o:`             /sssooo.   CPU: Intel Xeon X3430 @ 4x 2.395GHz
 /+++//+:`oo+o               /::--:.   GPU: ASPEED Technology, Inc. ASPEED Graphics Family (rev 10)
 \+/+o+++`o++o               ++////.   RAM: 472MiB / 7931MiB
  .++.o+++oo+:`             /dddhhh.  
       .+.o+oo:.          `oddhhhh+   
        \+.++o+o``-````.:ohdhhhhh+    
         `:o+++ `ohhhhhhhhyo++os:     
           .o:`.syhhhhhhh/.oo++o`     
               /osyyyyyyo++ooo+++/    
                   ````` +oo+++o\:    
                          `oo++.      
user@cisco-asa:~$ lscpu
Architecture:             x86_64
  CPU op-mode(s):         32-bit, 64-bit
  Address sizes:          36 bits physical, 48 bits virtual
  Byte Order:             Little Endian
CPU(s):                   4
  On-line CPU(s) list:    0-3
Vendor ID:                GenuineIntel
  Model name:             Intel(R) Xeon(R) CPU           X3430  @ 2.40GHz
    CPU family:           6
    Model:                30
    Thread(s) per core:   1
    Core(s) per socket:   4
    Socket(s):            1
    Stepping:             5
    Frequency boost:      enabled
    CPU(s) scaling MHz:   59%
    CPU max MHz:          2395.0000
    CPU min MHz:          1197.0000
    BogoMIPS:             4788.26
    Flags:                fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ht tm pbe syscall nx rdtscp lm constant_tsc arch_perfmon pebs bts rep_good nopl xtopology nonstop_tsc cpuid aperfmperf
                           pni dtes64 monitor ds_cpl vmx smx est tm2 ssse3 cx16 xtpr pdcm sse4_1 sse4_2 popcnt lahf_lm pti ssbd ibrs ibpb stibp tpr_shadow flexpriority ept vpid dtherm ida vnmi flush_l1d
Virtualization features:  
  Virtualization:         VT-x
Caches (sum of all):      
  L1d:                    128 KiB (4 instances)
  L1i:                    128 KiB (4 instances)
  L2:                     1 MiB (4 instances)
  L3:                     8 MiB (1 instance)
NUMA:                     
  NUMA node(s):           1
  NUMA node0 CPU(s):      0-3
Vulnerabilities:          
  Gather data sampling:   Not affected
  Itlb multihit:          KVM: Mitigation: VMX disabled
  L1tf:                   Mitigation; PTE Inversion; VMX conditional cache flushes, SMT disabled
  Mds:                    Vulnerable: Clear CPU buffers attempted, no microcode; SMT disabled
  Meltdown:               Mitigation; PTI
  Mmio stale data:        Unknown: No mitigations
  Reg file data sampling: Not affected
  Retbleed:               Not affected
  Spec rstack overflow:   Not affected
  Spec store bypass:      Mitigation; Speculative Store Bypass disabled via prctl
  Spectre v1:             Mitigation; usercopy/swapgs barriers and __user pointer sanitization
  Spectre v2:             Mitigation; Retpolines; IBPB conditional; IBRS_FW; STIBP disabled; RSB filling; PBRSB-eIBRS Not affected; BHI Not affected
  Srbds:                  Not affected
  Tsx async abort:        Not affected
  Vmscape:                Not affected
user@cisco-asa:~$ lsusb
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
Bus 001 Device 002: ID 8087:0020 Intel Corp. Integrated Rate Matching Hub
Bus 001 Device 003: ID 0624:0248 Avocent Corp. Virtual Hub
Bus 002 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
Bus 002 Device 002: ID 8087:0020 Intel Corp. Integrated Rate Matching Hub
Bus 002 Device 003: ID 1005:b155 Apacer Technology, Inc. Disk Module
user@cisco-asa:~$ lseth
lseth: command not found
user@cisco-asa:~$ lspci
00:00.0 Host bridge: Intel Corporation Core Processor DMI (rev 11)
00:03.0 PCI bridge: Intel Corporation Core Processor PCI Express Root Port 1 (rev 11)
00:05.0 PCI bridge: Intel Corporation Core Processor PCI Express Root Port 3 (rev 11)
00:08.0 System peripheral: Intel Corporation Core Processor System Management Registers (rev 11)
00:08.1 System peripheral: Intel Corporation Core Processor Semaphore and Scratchpad Registers (rev 11)
00:08.2 System peripheral: Intel Corporation Core Processor System Control and Status Registers (rev 11)
00:08.3 System peripheral: Intel Corporation Core Processor Miscellaneous Registers (rev 11)
00:10.0 System peripheral: Intel Corporation Core Processor QPI Link (rev 11)
00:10.1 System peripheral: Intel Corporation Core Processor QPI Routing and Protocol Registers (rev 11)
00:1a.0 USB controller: Intel Corporation 5 Series/3400 Series Chipset USB2 Enhanced Host Controller (rev 06)
00:1c.0 PCI bridge: Intel Corporation 5 Series/3400 Series Chipset PCI Express Root Port 1 (rev 06)
00:1c.4 PCI bridge: Intel Corporation 5 Series/3400 Series Chipset PCI Express Root Port 5 (rev 06)
00:1c.5 PCI bridge: Intel Corporation 5 Series/3400 Series Chipset PCI Express Root Port 6 (rev 06)
00:1d.0 USB controller: Intel Corporation 5 Series/3400 Series Chipset USB2 Enhanced Host Controller (rev 06)
00:1e.0 PCI bridge: Intel Corporation 82801 PCI Bridge (rev a6)
00:1f.0 ISA bridge: Intel Corporation 3450 Chipset LPC Interface Controller (rev 06)
00:1f.2 SATA controller: Intel Corporation 5 Series/3400 Series Chipset 6 port SATA AHCI Controller (rev 06)
00:1f.3 SMBus: Intel Corporation 5 Series/3400 Series Chipset SMBus Controller (rev 06)
01:00.0 PCI bridge: PLX Technology, Inc. PEX 8618 16-lane, 16-Port PCI Express Gen 2 (5.0 GT/s) Switch (rev ba)
02:01.0 PCI bridge: PLX Technology, Inc. PEX 8618 16-lane, 16-Port PCI Express Gen 2 (5.0 GT/s) Switch (rev ba)
02:03.0 PCI bridge: PLX Technology, Inc. PEX 8618 16-lane, 16-Port PCI Express Gen 2 (5.0 GT/s) Switch (rev ba)
02:05.0 PCI bridge: PLX Technology, Inc. PEX 8618 16-lane, 16-Port PCI Express Gen 2 (5.0 GT/s) Switch (rev ba)
02:07.0 PCI bridge: PLX Technology, Inc. PEX 8618 16-lane, 16-Port PCI Express Gen 2 (5.0 GT/s) Switch (rev ba)
02:09.0 PCI bridge: PLX Technology, Inc. PEX 8618 16-lane, 16-Port PCI Express Gen 2 (5.0 GT/s) Switch (rev ba)
02:0b.0 PCI bridge: PLX Technology, Inc. PEX 8618 16-lane, 16-Port PCI Express Gen 2 (5.0 GT/s) Switch (rev ba)
02:0d.0 PCI bridge: PLX Technology, Inc. PEX 8618 16-lane, 16-Port PCI Express Gen 2 (5.0 GT/s) Switch (rev ba)
02:0f.0 PCI bridge: PLX Technology, Inc. PEX 8618 16-lane, 16-Port PCI Express Gen 2 (5.0 GT/s) Switch (rev ba)
03:00.0 Ethernet controller: Intel Corporation 82574L Gigabit Network Connection
04:00.0 Ethernet controller: Intel Corporation 82574L Gigabit Network Connection
05:00.0 Ethernet controller: Intel Corporation 82574L Gigabit Network Connection
06:00.0 Ethernet controller: Intel Corporation 82574L Gigabit Network Connection
07:00.0 Ethernet controller: Intel Corporation 82574L Gigabit Network Connection
08:00.0 Ethernet controller: Intel Corporation 82574L Gigabit Network Connection
09:00.0 Ethernet controller: Intel Corporation 82574L Gigabit Network Connection
0a:00.0 Ethernet controller: Intel Corporation 82574L Gigabit Network Connection
0b:00.0 PCI bridge: PLX Technology, Inc. PEX 8624 24-lane, 6-Port PCI Express Gen 2 (5.0 GT/s) Switch [ExpressLane] (rev bb)
0c:04.0 PCI bridge: PLX Technology, Inc. PEX 8624 24-lane, 6-Port PCI Express Gen 2 (5.0 GT/s) Switch [ExpressLane] (rev bb)
0c:05.0 PCI bridge: PLX Technology, Inc. PEX 8624 24-lane, 6-Port PCI Express Gen 2 (5.0 GT/s) Switch [ExpressLane] (rev bb)
0c:08.0 PCI bridge: PLX Technology, Inc. PEX 8624 24-lane, 6-Port PCI Express Gen 2 (5.0 GT/s) Switch [ExpressLane] (rev bb)
0c:09.0 PCI bridge: PLX Technology, Inc. PEX 8624 24-lane, 6-Port PCI Express Gen 2 (5.0 GT/s) Switch [ExpressLane] (rev bb)
0f:00.0 Co-processor: Broadcom / LSI Device 0a05 (rev 01)
11:00.0 Network and computing encryption device: Cavium, Inc. CN15XX/CN16XX [Nitrox PX] (rev 01)
12:00.0 Ethernet controller: Intel Corporation 82574L Gigabit Network Connection
13:00.0 PCI bridge: ASPEED Technology, Inc. AST1150 PCI-to-PCI Bridge (rev 02)
14:00.0 VGA compatible controller: ASPEED Technology, Inc. ASPEED Graphics Family (rev 10)
ff:00.0 Host bridge: Intel Corporation Core Processor QuickPath Architecture Generic Non-Core Registers (rev 04)
ff:00.1 Host bridge: Intel Corporation Core Processor QuickPath Architecture System Address Decoder (rev 04)
ff:02.0 Host bridge: Intel Corporation Core Processor QPI Link 0 (rev 04)
ff:02.1 Host bridge: Intel Corporation Core Processor QPI Physical 0 (rev 04)
ff:03.0 Host bridge: Intel Corporation Core Processor Integrated Memory Controller (rev 04)
ff:03.1 Host bridge: Intel Corporation Core Processor Integrated Memory Controller Target Address Decoder (rev 04)
ff:03.2 Host bridge: Intel Corporation Core Processor Integrated Memory Controller Test Registers (rev 04)
ff:03.4 Host bridge: Intel Corporation Core Processor Integrated Memory Controller Test Registers (rev 04)
ff:04.0 Host bridge: Intel Corporation Core Processor Integrated Memory Controller Channel 0 Control Registers (rev 04)
ff:04.1 Host bridge: Intel Corporation Core Processor Integrated Memory Controller Channel 0 Address Registers (rev 04)
ff:04.2 Host bridge: Intel Corporation Core Processor Integrated Memory Controller Channel 0 Rank Registers (rev 04)
ff:04.3 Host bridge: Intel Corporation Core Processor Integrated Memory Controller Channel 0 Thermal Control Registers (rev 04)
ff:05.0 Host bridge: Intel Corporation Core Processor Integrated Memory Controller Channel 1 Control Registers (rev 04)
ff:05.1 Host bridge: Intel Corporation Core Processor Integrated Memory Controller Channel 1 Address Registers (rev 04)
ff:05.2 Host bridge: Intel Corporation Core Processor Integrated Memory Controller Channel 1 Rank Registers (rev 04)
ff:05.3 Host bridge: Intel Corporation Core Processor Integrated Memory Controller Channel 1 Thermal Control Registers (rev 04)
```