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

- 5V supply; choose a GOOD power supply to avoid brownout (XX: more details!)
- RPi4 , 4GB or 8GB
- Argone M.2 case (XX: more details!)
- SSD disk (XX: more details; comparible with M.2 Argone case)
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

- boot from SSD
- remove SD card altogether

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

## Sources

A number of sources I used for preparing this.

- https://github.com/sunknudsen/privacy-guides/blob/master/how-to-configure-hardened-raspberry-pi/README.md
- https://raspberrytips.com/security-tips-raspberry-pi/
- https://pimylifeup.com/raspberry-pi-security/
- https://piprojects.us/hardening-your-raspberry-pi-installation/
- https://github.com/tschaffter/raspberry-pi
- https://diode.io/raspberry%20pi/running-forever-with-the-raspberry-pi-hardware-watchdog-20202/
- 
