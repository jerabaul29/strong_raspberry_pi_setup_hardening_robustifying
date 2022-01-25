# strong_rpi_setup

A general setup for a strong Raspberry Pi. I want a setup that is strong enough that I can make it available to the internet, SSH safely to it, trust that it has good uptime etc. This readme should describe how to achieve this, through:

- using "good" hardware
- using encryption of the hard drive
- changing all the necessary defaults to avoid out-of-the-box default passwords and accounts and hostnames, 
- setting up a robust SSH configuration
- setting up the watchdog
- setting up some reverse tunnel to a trusted server

## Hardware

## General setup

### SD card

### boot from SSD

### encrypted data partition

- decode by sending password through ssh?

## Robustifying

### Change user account name and password

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
