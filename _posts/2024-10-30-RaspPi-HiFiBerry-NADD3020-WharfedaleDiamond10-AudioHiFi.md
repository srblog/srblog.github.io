---
layout: post
title:  "RaspPi-LMS + HiFiBerry + NAD-D3020 + Wharfedale-Diamond = Sweet Audio Rig"
date:   2024-10-30
tags: RaspPi Music Audio HiFi 
---
# HiFi Setup 

![HiFi Setup](/assets/images/2024-10-30-hifiSetup.png)

This audio rig is for an audio enthusiast looking to step into the world of audiophile on a budget.
This setup consists of four components: the music server (RasPi+LMS), the audio playback unit (RasPi+HiFiBerry), the amplifier (NAD D3020), and the speakers (Wharfedale Diamond 10). 
Currently, the entire setup can be put together for under USD 1,000.

# Setting up Raspberry Pi as NAS Server

**INSTALLING RASPBERRY PI OS LITE**

- Download and start the Raspberry Pi imager.
- Choose the OS type: `Raspberry Pi OS Lite (64 bit)`
- Choose the target SD card.
- From _settings_ preconfigure avaialble options eg. `ssh creds, WiFi creds, hostname, etc.`. This is essential for _headless_ install.
- Login using PuTTy using the preconfigured SSH user and update OS and install essential eg. `vim`

**INSTALLING UFW FIREWALL (OPTIONAL)**

NOT NEEDED if unit is inside a firewall.

- `sudo apt install ufw`
- Before rebooting make sure you allow SSH: `sudo ufw allow 22`
- Check the status `sudo ufw status`

**MOUNTING EXTERNAL SSD**

- Find the block device name eg. `/dev/sda` using the command `lsblk`
- Create a partition (not necessary but highly recommended) using `fdisk`:
  - `fdisk -l` to list all partitions available.
  - `fdisk /dev/sda`
  - `n` for creating a partition. For creating one Linux partition use the defaults.
  - `p` print the partition for checking.
  - `w` write the partition to disk.
- Now you can see the block device for the new partition eg. `/dev/sda1` using `lsblk`
- Create a `ext4` partition: `sudo mkfs.ext4 /dev/sda1`
- Find the _UUID_ of the partition: `blkid /dev/sda1`
- Create a mount point: `sudo mkdir /media/wd220`
- Create a fstab entry in `/etc/fstab`:
  - `UUID=5abdf860-950b-40b8-8799-49f6ce70044c /media/wd220 ext4 defaults,auto,users,rw,nofail 0 0` 
  - **FIXME** document the options.
- `sudo mount -a` 

**CREATING A SAMBA SHARE**

Could not get a Public share to work. Write permission error from Windows. After trying out lot of ways, following seem to work for a Private share.

- Create the share directory: `sudo mkdir /media/wd220/Music`
- Change owner, group and permission to the user that will be a samba user say smbuser:
  - `sudo chown smbuser /media/wd220/Music`
  - `sudo chgrp users /media/wd220/Music`
  - `sudo chmod 2775 /media/wd220/Music` The 2 in the begining makes the folder sticky so users in the group "users" can write to the directory with their ownership.
- Install _samba_ : `sudo apt-get install samba smb-client cifs-utils`
- Add the Windows group (eg. WORKGROUP) to the global option in `/etc/samba/smb.conf`
- Add the share folder to `/etc/samba/smb.conf`:

{% highlight bash %}
[Music]
   comment = Public Music Folder
   path = /media/wd220/Music
   read only = no
   guest ok = no
   valid users = smbuser
{% endhighlight %}

- `sudo smbpasswd -a smbuser` Add the user as a Samba user. You are going to use this credential when accessing the folder from Windows.
- Allow the SMB ports 139,445 in the firewall (if ufw enabled):

# LOGITECH MEDIA SERVER (LMS) on RaspberryPiOS

This section documents the steps in installing **Logitech Media Server** for organizing Music files. Also, this installation was done on the Raspberry Pi OS that is already running OpenMediaVault (OMV) . So all filesystems are managed through OMV.

Followed the LMS part of [Harald Kreuzer's Blog](https://www.haraldkreuzer.net/en/news/installing-logitech-media-server-raspberry-pi-4b-5-inch-display). **NOTE** There is section in the blog about disabling the swap to increase the longetivity of the SD card. Worth looking into it.

Important steps are listed below:

- `sudo apt-get update && sudo apt-get upgrade -y` followed by `sudo reboot`
- **Install LMS Server**:

{% highlight bash %}
sudo apt-get install libsox-fmt-all libflac-dev libfaad2 
sudo apt-get install libio-socket-ssl-perl 
sudo apt-get install libcrypt-openssl-bignum-perl 
sudo apt-get install libcrypt-openssl-random-perl 
sudo apt-get install libcrypt-openssl-rsa-perl 
wget https://downloads.slimdevices.com/LogitechMediaServer_v8.3.1/logitechmediaserver_8.3.1_arm.deb 
sudo dpkg -i logitechmediaserver_8.3.1_arm.deb
{% endhighlight %}

**NOTE** Check the latest LMS distro available.

- `sudo ufw allow 9000`
- The server will now be accessible at `http://<IP>:9000/`
  - **NOTE** you do not need to create the logitech account. You can skipt.
- **FIXME** Create the appropriate directory structure in OMV and configure that here.
  - You can check these links ([BegineersGuide](https://wiki.slimdevices.com/index.php/Beginners_Guide_To_Organising.html), [Survey](http://www.hydrogenaudio.org/forums/index.php?showtopic=32726), [Organize4DJ-MP3Tags](https://homedjstudio.com/organize-music-library/)) to see some popular ways to organize folders for music.
  - One popular choice is: `/<library>/<artist>/<album>/<tracks>`
  - Organizing Classical Music is different (See [BegineersGuideToClassical](https://wiki.slimdevices.com/index.php/BeginnersGuideToClassical.html). A simple _tagging_ (Tagged during ripping) reco from the guide:
    - the folder strategy I am using now is: `/<lib>/<composer>/<CDalbum>/<tracks>`
    - For Various artiests: `/<lib>/<VariousArtist>/<CDalbum>/<tracks>`
    - **Tag** reco:
      - _Album_ tag for _Work_ eg. "Beethoven Symphony no. 5 - Karajan"
      - _Artist_ tag for _Composer_ eg. "Beethoven"
      - _Title_ tag for _Movement_ eg. "Beethoven Symp 5 - 3- Allegro"
- You can remove all unnecessary plugins. 
- It's worth installing the `material` plugin which is a responsive plugin so the server will be accessible at `https://<IP>:9000/material` even from a mobile.

## AWS CLI

- Check this [section](https://mixignal.github.io/wiki/compute-it.html?highlight=aws%20cli#setting-up-a-linux-vm) for installing aws cli tools.
- **Note**, download the ARM version instead of the x86 version: 
  - `curl "https://awscli.amazonaws.com/awscli-exe-linux-aarch64.zip" -o "awscliv2.zip"`

**Setting crontab for syncing from AWS S3**

- `00 03 * * 1 aws s3 sync --delete --profile srout s3://linode-vm01-bak /media/wd220/aws-S3/srout/linode-vm01-bak > /home/srout/logs/s3sync-linodevm01.log 2>&1`

{% highlight bash %}
sudo ufw allow 139
sudo ufw allow 445
{% endhighlight %}

- `sudo systemctl restart smbd` Restart the SMB daemon.
- Now try accessing from Windows `Win+R` and entering `\\[IP/hostname]/Music`
- Some useful links: [devconnected](https://devconnected.com/how-to-install-samba-on-debian-10-buster/#:~:text=In%20order%20for%20Samba%20to,on%20ports%20139%20and%20445.) | [ComputingForGeeks](https://computingforgeeks.com/how-to-configure-samba-share-on-debian/?expand_article=1) | [ServerSpace](https://serverspace.io/support/help/configuring-samba-on-debian/)



# Setting Up HiFiBerry

(**FIXME** Content)


# References 

- Oracle, “Chapter 1 Introduction to the Solaris X Server.” Accessed: Oct. 02, 2023. [Online](https://docs.oracle.com/cd/E19455-01/806-1363/6jalfckmd/index.html)
