# strong_rpi_setup headless

A general setup for a strong Raspberry Pi headless setup. I want a setup that is strong enough that I can make it available to the internet, SSH safely to it, trust that it has good uptime etc. This readme should describe how to achieve this, through:

- using "good" hardware
- setting things up for headless use
- using encryption of the hard drive
- changing all the necessary defaults to avoid out-of-the-box default passwords and accounts and hostnames, 
- setting up a robust SSH configuration
- setting up the watchdog
- setting up some reverse tunnel to a trusted server

## Hardware

## General setup

Note that it may be a good idea to buy everything from the same shop to limit shipping costs; for example, https://thepihut.com use to have a good sample of products.

- 5V supply; choose a GOOD power supply to avoid brownout (for example, the official one with the right plug: https://www.raspberrypi.com/products/micro-usb-power-supply/?variant=raspberry-pi-125w-psu-eu-wh)
- RPi4 , 4GB or 8GB
- Argone M.2 case (https://www.argon40.com/argon-one-m-2-case-for-raspberry-pi-4.html)
- M.2 SATA SSD disk (do NOT buy a NVMe disk, you need a M.2 SATA; for example, https://thepihut.com/products/wd-green-240gb-m-2-internal-ssd or similar; also, check out the max power consumption allowed for the drive by what the RPi can supply, vs. the consumption of the SSD drive, and make sure there is a bit of margin; if not, the SSD drive may not work or be unstable)
- SD card (8GB or 16GB will be fine)

### SD card

Prepare the SD card using the rpi-image tool:

```
~$ rpi-imager
```

And follow instructions. If you are using a Rpi with less or equal to 4GB, both 32 and 64 bit OSes are ok; if you are using a RPi with more than 4GB, you will need a 64 bit OS to take advantage of that - a 32 bit OS will work, but only up to around 4GB or RAM will actually be used.

As always, remember to mount the SD card when you insert it and want to access it, and unmount / eject the SD card before removing it.

In order to be able to SSH into the RPi the first time you boot it, remember to add a file called ```ssh``` in the boot partition of the SD card:

```
/media/jrmet/boot$ touch ssh
```

Without this file, SSH connection will be disabled as a security measure.

### Assembly

There are a number of nice cases, for my part, I like best the Argon ONE case (for providing a nice housing and cooling), or Argon ONE M.2 (for providing housing, cooling, and place for a SSD-over-USD drive). Remember that the format between RPi up to 3 vs. RPi 4 are different, so make sure to use a case that is compatible with your model of RPi.

Assemble following the instructions, make sure to:

- avoid touching the PCBs with your fingers due to the risk of electrostatic discharge
- insert the SD card only when you are finished putting the PCBs and components together
- remember to put the thermal paste
- assemble the M.2 drive and screw it in place

### Initial boot and SSHing headless

Power the RPi and connect a Ethernet cable between the RPi and your machine.

Set up the network on your machine so that it is possible to establish connection between the RPi and your machine over Ethernet. For example, on Ubuntu:

```
- Go to the network manager (top left)
- Enter Wired Settings
- Create a new profile
- Use the following parameters:
  - connect automatically
  - make available to other usrs
  - no metered connection
  - name: RPi
  - MTU: automatic
  - IPv4: shated to other computers
  - IPv6: shared to other computers
  - Security: none
```

Find the IP range on the interface on which the RPi is connected using ifconfig; in my case, I am connected on ethernet, so:

```
~$ ifconfig
[...]
enp0s31f6: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.42.0.1  netmask 255.255.255.0  broadcast 10.42.0.255
[...]
```

Scan for the RPi to find its address:

```
~$ nmap -sP 10.42.0.0/24
Starting Nmap 7.80 ( https://nmap.org ) at 2022-01-30 11:30 CET
Nmap scan report for L590 (10.42.0.1)
Host is up (0.00076s latency).
Nmap scan report for 10.42.0.208
Host is up (0.00028s latency).
Nmap done: 256 IP addresses (2 hosts up) scanned in 2.85 seconds
```

SSH into the RPi using the IP address you just found; default username: pi ; default password: raspberry . Note that to avoid being banished due to too many failed attempts you may need to ask for password authentication, so that your computer does not try all the SSH you own before trying to password connect, which may be blocked due to too many attempts:

```
~$ ssh -o PreferredAuthentications=password pi@10.42.0.208
pi@10.42.0.208's password: 
[...]
pi@raspberrypi:~ $ 
```

Change the password at once:

```
pi@raspberrypi:~ $ passwd
for example to something like, or more complicated than: some_new_password_55
```

At this point, update the RPi:

```
sudo apt update
sudo apt upgrade
sudo apt autoremove
```

The next thing is to configure the RPi:

```
sudo raspi-config
Interface Option > SSH > enable SSH server: yes
Localization Option > locale > 
```

If you are using an Argone housing, install the corresponding management tools:

```
pi@raspberrypi:~ $ curl https://download.argon40.com/argon1.sh > argon1.sh
pi@raspberrypi:~ $ cat argon1.sh # if you want to inspect what it does
argonne-config
```

Choosing "decent" defaults, the annoying fan noise should be gone except maybe when using the CPU at maximum capacity.

### boot from SSD

Make sure that the M.2 SATA SSD disk is well installed, and that the USB connector outside of the box is well in place to connect the RPi to the SSD disk.

Then:

- full update the RPi:

```
sudo apt-get update
sudo apt-get full-upgrade
sudo rpi-update
sudo reboot
sudo rpi-eeprom-update -d -a
sudo reboot
```

And use raspi-config to use the latest ROM for boot:

```
sudo raspi-config
# > Advanced options > Bootloader Version > E1 Latest
# check what is displayed then
```

- check that the SSD is well present and visible; if it is, you will see both the SD card and the SSD:

```
pi@raspberrypi:~ $ lsusb
Bus 002 Device 002: ID 174c:1156 ASMedia Technology Inc. Forty
Bus 002 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
Bus 001 Device 002: ID 2109:3431 VIA Labs, Inc. Hub
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
```

The ASMedia entry is the SSD disk; it is also visible as:

```
pi@raspberrypi:~ $ lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda           8:0    0  1.8T  0 disk 
mmcblk0     179:0    0 14.8G  0 disk 
|-mmcblk0p1 179:1    0  256M  0 part /boot
`-mmcblk0p2 179:2    0 14.6G  0 part /
```

See the sda 1.8T entry.

- At this point, we need to clone all the system from the SD card to the SSD disk. Since we work headless, we need to use a script to make the SSD bootable instead of using the "usual" GUI tool: see https://github.com/billw2/rpi-clone . To install rpi-clone:

```
pi@raspberrypi:/ $ cd /home/pi/
pi@raspberrypi:~ $ mkdir Git
pi@raspberrypi:~ $ cd Git/
pi@raspberrypi:~/Git $ git clone https://github.com/billw2/rpi-clone.git
pi@raspberrypi:~/Git $ cd rpi-clone/
pi@raspberrypi:~/Git/rpi-clone $ sudo cp rpi-clone rpi-clone-setup /usr/local/sbin
pi@raspberrypi:~/Git/rpi-clone $ sudo rpi-clone-setup -t PUT_YOUR_HOSTNAME
```

- Next, we need to actually run rpi-clone. For this, we need to find the destination on which to clone, and run the command. This will both clone the boot and the "normal" partitions, and resize them to take advantage of the full disk. Note that this can take a few minutes.

```
pi@raspberrypi:~/Git/rpi-clone $ lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda           8:0    0  1.8T  0 disk 
mmcblk0     179:0    0 14.8G  0 disk 
|-mmcblk0p1 179:1    0  256M  0 part /boot
`-mmcblk0p2 179:2    0 14.6G  0 part /
pi@raspberrypi:~/Git/rpi-clone $ sudo rpi-clone sda
Error: /dev/sda: unrecognised disk label

Booted disk: mmcblk0 15.9GB                Destination disk: sda 2.0TB
---------------------------------------------------------------------------
Part      Size    FS     Label           Part   Size  FS  Label  
1 /boot   256.0M  fat32  --                                      
2 root     14.6G  ext4   rootfs                                  
---------------------------------------------------------------------------
== Initialize: IMAGE partition table - partition number mismatch: 2 -> 0 ==
1 /boot               (48.5M used)   : MKFS  SYNC to sda1
2 root                (3.6G used)    : RESIZE  MKFS  SYNC to sda2
---------------------------------------------------------------------------
Run setup script       : no.
Verbose mode           : no.
-----------------------:
** WARNING **          : All destination disk sda data will be overwritten!
-----------------------:

Initialize and clone to the destination disk sda?  (yes/no): yes
Optional destination ext type file system label (16 chars max): 

Initializing
  Imaging past partition 1 start.
  => dd if=/dev/mmcblk0 of=/dev/sda bs=1M count=8 ...
  Resizing destination disk last partition ...
    Resize success.
  Changing destination Disk ID ...
  => mkfs -t vfat -F 32  /dev/sda1 ...
  => mkfs -t ext4  /dev/sda2 ...

Syncing file systems (can take a long time)
Syncing mounted partitions:
  Mounting /dev/sda2 on /mnt/clone
  => rsync // /mnt/clone with-root-excludes ...
  Mounting /dev/sda1 on /mnt/clone/boot
  => rsync /boot/ /mnt/clone/boot  ...

Editing /mnt/clone/boot/cmdline.txt PARTUUID to use 3e66da39
Editing /mnt/clone/etc/fstab PARTUUID to use 3e66da39
===============================
Done with clone to /dev/sda
   Start - 17:03:37    End - 17:06:25    Elapsed Time - 2:48

Cloned partitions are mounted on /mnt/clone for inspection or customizing. 

Hit Enter when ready to unmount the /dev/sda partitions ...
  unmounting /mnt/clone/boot
  unmounting /mnt/clone
===============================
```

- To inspect the content of hte newly cloned SSD disk, see:

```
pi@raspberrypi:/ $ sudo mkdir /mnt/sda_disk_1
pi@raspberrypi:/ $ sudo mkdir /mnt/sda_disk_2
pi@raspberrypi:/ $ sudo mount /dev/sda1 /mnt/sda_disk_1/
pi@raspberrypi:/ $ sudo mount /dev/sda2 /mnt/sda_disk_2/
```

and you can look at ```/mnt/sda_disk_1/``` and ```_2``` and check that all looks good.

- set the boot order:

```
sudo raspi-config
# > Advanced options > Boot order > B2 USB boot and ENTER to confirm
# note that a reboot will be needed for this to take effect; you can wait to reboot though, as we need to set up the SSD disk
```

Note: if something goes wrong in the following, the RPi will fail to boot, as it will try to boot from the wrongly copied / empty SSD disk; to remedy this, in case the SSD disk has not been cloned yet or something went wrong but you want to reboot, simply remove the USB jumper before rebooting, and the SSD disk will not be available, so that the RPi will default to booting on the SD card instead.

- At this stage, we should have 1) by default, boot from the USB SSD drive external storage, 2) cloned both partitions of the SD card into the SSD drive to make it bootable and usable as the main drive. So, we are ready to reboot, and the boot will happen from the SSD drive this time. This is confirmed by (note that booting from the SSD the first time may take a few seconds extra):

```
pi@raspberrypi:~ $ sudo reboot now
~$ ssh -o PreferredAuthentications=password pi@10.42.0.208
pi@10.42.0.208's password: 
pi@raspberrypi:~ $ df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/root       1.8T  3.7G  1.7T   1% /
devtmpfs        3.7G     0  3.7G   0% /dev
tmpfs           3.9G     0  3.9G   0% /dev/shm
tmpfs           1.6G  944K  1.6G   1% /run
tmpfs           5.0M  4.0K  5.0M   1% /run/lock
/dev/sda1       253M   49M  204M  20% /boot
tmpfs           790M   24K  790M   1% /run/user/1000
/dev/mmcblk0p2   15G  3.7G   11G  27% /media/pi/rootfs
/dev/mmcblk0p1  253M   49M  204M  20% /media/pi/boot
```

You can see here that ```dev/root``` is now 1.8T in size, i.e we are well booted to the SSD disk, and the 2 partitions of the SD card are visible as ```/dev/mmcblk0p1``` and ```2```: all is in good order.

## Robustifying

- what threat model?
- threat model is not physical access; nothing to do against it (or nearly, or that would be a lot of work)

### encrypted data partition

- if physical access: ok to have full disk encryption
- if not: what is threat model? is it just that the SSD hard drive gets thrown away later on?
- if no physical access: need to be able to decrypt data; ok in case the RPi is protected and cannot be tampered with, but still want to protect very sensitive data just in case the hard disk is remobed at some point
- encrypted data partition on the SSD; use the SD card to store the key to decrypt it; so, when throwing stuff away, as long as SD card and SSD do not end up at the same place, will be useless
- does not protect against physical access, but what would? ...
- TPM could be a better alternative of course :)

### Change user account name and password

### admin session, user session, dummy ssh tunnel session

### Change hostname

### Disable SSH password logging

### Set 2FA on SSH

### Change SSH port

### Use UFW to limit SSH attempts

### Ban IPs

### Setup reverse SSH tunnel to a trusted server

### Disable root loging and set root password

## Ensure uptime and up to date

### Keep updated

### Set up watchdog

### Set up network watchdog

### Send emails with status to admin

### disable 

### if boot from SD card, send notice about it, and reboot instead

## Sources

A number of sources I used for preparing this.

- https://github.com/sunknudsen/privacy-guides/blob/master/how-to-configure-hardened-raspberry-pi/README.md
- https://raspberrytips.com/security-tips-raspberry-pi/
- https://pimylifeup.com/raspberry-pi-security/
- https://piprojects.us/hardening-your-raspberry-pi-installation/
- https://github.com/tschaffter/raspberry-pi
- https://diode.io/raspberry%20pi/running-forever-with-the-raspberry-pi-hardware-watchdog-20202/
- 
