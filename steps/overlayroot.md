# Overlay RootFS with tmpfs to prevent sd card wear

Raspberry Pi OS (Debian) will keep writing logs to the sd card causing wear, among other possible issues. This solution overlays the root filesystem with tmpfs so that all writes happen to RAM and the disk remains read-only.

## Make an init shell script

Create a file called `/sbin/overlayRoot.sh` with the following contents:

```
#!/bin/sh
fail(){
	echo -e "$1"
	/bin/bash
}
 
# load module
modprobe overlay
if [ $? -ne 0 ]; then
    fail "ERROR: missing overlay kernel module"
fi
# mount /proc
mount -t proc proc /proc
if [ $? -ne 0 ]; then
    fail "ERROR: could not mount proc"
fi
# create a writable fs to then create our mountpoints 
mount -t tmpfs inittemp /mnt
if [ $? -ne 0 ]; then
    fail "ERROR: could not create a temporary filesystem to mount the base filesystems for overlayfs"
fi
mkdir /mnt/lower
mkdir /mnt/rw
mount -t tmpfs root-rw /mnt/rw
if [ $? -ne 0 ]; then
    fail "ERROR: could not create tempfs for upper filesystem"
fi
mkdir /mnt/rw/upper
mkdir /mnt/rw/work
mkdir /mnt/newroot
# mount root filesystem readonly 
rootDev=`awk '$2 == "/" {print $1}' /etc/fstab`
rootMountOpt=`awk '$2 == "/" {print $4}' /etc/fstab`
rootFsType=`awk '$2 == "/" {print $3}' /etc/fstab`
echo "check if we can locate the root device based on fstab"
blkid $rootDev
if [ $? -gt 0 ]; then
    echo "no success, try if a filesystem with label 'rootfs' is avaialble"
    rootDevFstab=$rootDev
    rootDev=`blkid -L "rootfs"`
    if [ $? -gt 0 ]; then
        echo "no luck either, try to further parse fstab's root device definition"
        echo "try if fstab contains a PARTUUID definition"
        echo "$rootDevFstab" | grep 'PARTUUID=\(.*\)-\([0-9]\{2\}\)'
        if [ $? -gt 0 ]; then 
	    fail "could not find a root filesystem device in fstab. Make sure that fstab contains a device definition or a PARTUUID entry for / or that the root filesystem has a label 'rootfs' assigned to it"
        fi
        device=""
        partition=""
        eval `echo "$rootDevFstab" | sed -e 's/PARTUUID=\(.*\)-\([0-9]\{2\}\)/device=\1;partition=\2/'`
        rootDev=`blkid -t "PTUUID=$device" | awk -F : '{print $1}'`p$(($partition))
        blkid $rootDev
        if [ $? -gt 0 ]; then
	    fail "The PARTUUID entry in fstab could not be converted into a valid device name. Make sure that fstab contains a device definition or a PARTUUID entry for / or that the root filesystem has a label 'rootfs' assigned to it"
        fi
    fi
fi
mount -t ${rootFsType} -o ${rootMountOpt},ro ${rootDev} /mnt/lower
if [ $? -ne 0 ]; then
    fail "ERROR: could not ro-mount original root partition"
fi
mount -t overlay -o lowerdir=/mnt/lower,upperdir=/mnt/rw/upper,workdir=/mnt/rw/work overlayfs-root /mnt/newroot
if [ $? -ne 0 ]; then
    fail "ERROR: could not mount overlayFS"
fi
# create mountpoints inside the new root filesystem-overlay
mkdir /mnt/newroot/ro
mkdir /mnt/newroot/rw
# remove root mount from fstab (this is already a non-permanent modification)
grep -v "$rootDev" /mnt/lower/etc/fstab > /mnt/newroot/etc/fstab
echo "#the original root mount has been removed by overlayRoot.sh" >> /mnt/newroot/etc/fstab
echo "#this is only a temporary modification, the original fstab" >> /mnt/newroot/etc/fstab
echo "#stored on the disk can be found in /ro/etc/fstab" >> /mnt/newroot/etc/fstab
# change to the new overlay root
cd /mnt/newroot
pivot_root . mnt
exec chroot . sh -c "$(cat <<END
# move ro and rw mounts to the new root
mount --move /mnt/mnt/lower/ /ro
if [ $? -ne 0 ]; then
    echo "ERROR: could not move ro-root into newroot"
    /bin/bash
fi
mount --move /mnt/mnt/rw /rw
if [ $? -ne 0 ]; then
    echo "ERROR: could not move tempfs rw mount into newroot"
    /bin/bash
fi
# unmount unneeded mounts so we can unmout the old readonly root
umount /mnt/mnt
umount /mnt/proc
umount /mnt/dev
umount /mnt
# continue with regular init
exec /sbin/init
END
)"
```

### Set permissions

```
chmod 755 /sbin/overlayRoot.sh
```

## Turn off swap

```
dphys-swapfile swapoff
dphys-swapfile uninstall
update-rc.d dphys-swapfile remove
systemctl disable dphys-swapfile
```

## Enable overlayRoot

This is achieved by adding ` init=/sbin/overlayRoot.sh` to the end of the commnad line in `/boot/cmdline.txt`, and rebooting.

Below one-liner can do it for you:

```
grep overlayRoot.sh /boot/cmdline.txt || sed -i 's/$/ init=\/sbin\/overlayRoot.sh/g' /boot/cmdline.txt ; reboot
```

## Disable overlayRoot

Yes, you can reverse this easily to make changes. Just remove ` init=/sbin/overlayRoot.sh` from the end of `/boot/cmdline.txt` and reboot. The below script will do it for you:

```
sed -i 's/ init=\/sbin\/overlayRoot.sh//g' /boot/cmdline.txt ; reboot
```
