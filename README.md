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

- 5V supply
- RPi4 8GB
- Argone M.2 case
- SSD disk

### SD card

- small SD card

### Assembly

- assemble

### Initial boot

- SD card preparation
- boot

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
