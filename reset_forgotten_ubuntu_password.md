# reset forgotten ubuntu password

## go to 
    – advance option for ubuntu
    – dpkg
    – yes
    – d
```sh
!/bin/bash
```

## then
```sh
passwd username
```
## set the password and reboot
```sh
reboot
```

# previously the way was
## go to 
    – advance option for ubuntu
    – root

```sh
mount -o remount,rw /
``` 
## then
```sh
passwd username
```
## set the password and reboot
```sh
reboot   