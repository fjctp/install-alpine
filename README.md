# install-alpine

Setup Alpine Linux in diskless mode and use local backup tool, `lbu` to preserve custom configs and package selections.

## Install

Assumptions:

* Downloaded the latest `extended` iso from [link](https://alpinelinux.org/downloads/).
* Booted from the image either on virtual machine or hardware.
* `/dev/vda` is the target drive and has 2 partitations
  * `vda1`: system disk. Must format to `FAT32`
  * `vda2`: custom configs and packages. 

1. Copy Alpine's files to `/dev/vda1`

``` bash
setup-alpine # For the last 3 questions related to disk, answer "none"
             # the other questions do not matter.
mount /dev/vda1 /media/vda1
setup-bootable -v /media/cdrom/ /media/vda1
reboot # rebiit ti newky installed system
```

2. Setup local backup tool: `lbu`

``` bash
setup-alpine # answer ALL questions properly
             # for the last 3 question related to disks: 
             #   1. none # no data disk
             #   2. vda2 # use /dev/vda2 for custom configs
             #   3. /media/vda2/cache/apk # save packages to this location
echo "BACKUP_LIMIT=3" >> /etc/lbu/lbu.conf # keep last three backups

lbu commit # remember to save the settings
reboot
```

Ensure that settings persisted after a reboot, like hostname

3. use UUID for mounting [optional]

``` bash
blkid | grep -i /dev/vda2 | awk {print $3} | tee >> /etc/fstab
vim /etc/fstab # update it accordingly
```

Note: if not mounting `/media/vda2` anymore, remember update the settings

``` bash
setup-lbu [NEW_MEDIA_LOCATION] # see -help
setup-apkcache [NEW_LOCATION]
```

4. add user: test

``` bash
addgroup test
adduser -G test test
adduser -G users test
adduser -G wheel test

lbu add /home # add /home to the backup list
lbu commit # remember to save the settings
```

5. install `doas`, lightweight version of `sudo`

``` bash
apk add doas
echo "permit persist :wheel" >> /etc/doas.conf

lbu commit # remember to save the settings
```

## Notes

* custom configs are packed up and save to `/media/vda1/*.apkovl.tar.gz`
* package selection are kept in `/etc/apk/world` and the package cache are located in `/etc/apk/cache`. If packages don't presist between boot, make sure `lbu` and `apkcache` have the correct settings
