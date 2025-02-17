> [!CAUTION]
> READ CAREFULLY! When using Chroot environments to exit completely close the Termux application even from the background apps or if necessary force close it. Otherwise, in case you do some command like "rm -rf chrootFolder" the device will go crazy and you will have to force reboot it.

##  First steps <a name=first-steps-chroot></a>

1. First you need to have your device <u>rooted</u>.
2. You need to flash [Busybox](https://github.com/Magisk-Modules-Alt-Repo/BuiltIn-BusyBox/releases) with Magisk.
3. Then you need to install the following packages in Termux: 

```
pkg update
pkg install root-repo
pkg update
pkg install tsu
```


---  
<br>

## Setting Ubuntu chroot <a name=ubuntu-chroot></a>

1. Enter Android shell with root privileges: 
```
su
```

2. Create a directory at `/data/local/tmp` for chroot environment
```
mkdir /data/local/tmp/ubuntuchroot
cd /data/local/tmp/ubuntuchroot
```

3. Download [Ubuntu 22.0.4 LTS](https://cdimage.ubuntu.com/ubuntu-base/releases/22.04/release/) rootfs: 
```
curl https://cdimage.ubuntu.com/ubuntu-base/releases/22.04/release/ubuntu-base-22.04-base-arm64.tar.gz --output ubuntu.tar.gz

```

4. Unzip the downloaded file and create some folders to mount the sdcard
```
tar xpvf ubuntu.tar.gz --numeric-owner
```

5. Create a start script: 
```
cd ../
nano start.sh
```
Copy and paste the following: 
```
#!/bin/sh

# The path of Ubuntu rootfs
UBUNTUPATH="/data/local/tmp/ubuntuchroot"

# Fix setuid issue
busybox mount -o remount,dev,suid /data

busybox mount --bind /dev $UBUNTUPATH/dev
busybox mount --bind /sys $UBUNTUPATH/sys
busybox mount --bind /proc $UBUNTUPATH/proc
busybox mount -t devpts devpts $UBUNTUPATH/dev/pts

# /dev/shm for Electron apps
mkdir $UBUNTUPATH/dev/shm
busybox mount -t tmpfs -o size=256M tmpfs $UBUNTUPATH/dev/shm

# Mount sdcard
mkdir $UBUNTUPATH/sdcard
busybox mount --bind /sdcard $UBUNTUPATH/sdcard

# chroot into Ubuntu
busybox chroot $UBUNTUPATH /bin/su - root
```

6. Make the script executable and run it: 
```
chmod +x start.sh
sh start.sh
```

7. The prompt will change to `root@localhost`. If you need to return to Termux just write `exit`. Let's execute some fixes: 
```
echo "nameserver 8.8.8.8" > /etc/resolv.conf
echo "127.0.0.1 localhost" > /etc/hosts

groupadd -g 3003 aid_inet
groupadd -g 3004 aid_net_raw
groupadd -g 1003 aid_graphics
usermod -g 3003 -G 3003,3004 -a _apt
usermod -G 3003 -a root

apt update
apt upgrade

apt install nano vim net-tools sudo git
```

8. Setup timezone: 
```
ln -sf /usr/share/zoneinfo/Europe/Minsk /etc/localtime
```

9. Create a new user called `linuxonadrod` (or the name you prefer)
```
groupadd storage
groupadd wheel
useradd -m -g users -G wheel,audio,video,storage,aid_inet -s /bin/bash linuxonadrod
passwd linuxonadrod
```

10. Add the created user to sudoers file to have superuser privileges: 
```
nano /etc/sudoers
```
Add this line: 
```
linuxonadrod ALL=(ALL:ALL) ALL
```

11. Switch to the created user: 
```
sudo apt install locales
sudo locale-gen en_US.UTF-8
```

> [!NOTE]
> This step is for Ubuntu only
12. Disable Snapd (it can't be used on Termux): 
```
apt-get autopurge snapd

cat <<EOF | sudo tee /etc/apt/preferences.d/nosnap.pref
# To prevent repository packages from triggering the installation of Snap,
# this file forbids snapd from being installed by APT.
# For more information: https://linuxmint-user-guide.readthedocs.io/en/latest/snap.html
Package: snapd
Pin: release a=*
Pin-Priority: -10
EOF
```

# Done!

* Installing NerdFonts on chroot env (Debian, Ubuntu) from this [post](https://medium.com/@almatins/install-nerdfont-or-any-fonts-using-the-command-line-in-debian-or-other-linux-f3067918a88c):
```
sudo apt install wget unzip -y
```
```
wget -P ~/.local/share/fonts https://github.com/ryanoasis/nerd-fonts/releases/download/v3.0.2/JetBrainsMono.zip \
&& cd ~/.local/share/fonts \
&& unzip JetBrainsMono.zip \
&& rm JetBrainsMono.zip \
&& fc-cache -fv
```
