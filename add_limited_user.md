# Add a limited user

- Add the user

```
pi@raspberrypi:~ $ sudo adduser limited_user
# adding the new password: some_new_password
```

- To give a dummy shell that lets do nothing: edit /etc/passwd to change the user shell to:

```
/usr/sbin/nologin
```

This will display:

```
~$ ssh -o PreferredAuthentications=password limited_user@10.42.0.208
limited_user@10.42.0.208's password: 
Linux raspberrypi 5.10.94-v7l+ #1518 SMP Thu Jan 27 14:54:16 GMT 2022 armv7l

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Sun Mar  6 20:39:22 2022 from 10.42.0.1
This account is currently not available.
Connection to 10.42.0.208 closed.
```
