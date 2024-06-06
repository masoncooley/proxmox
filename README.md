# proxmox

## Installation
I first grabbed the Proxmox Virtual Environment 8.2.2 `.iso` from the official website and loaded it onto a Ventoy USB drive. While trying to boot from the installation drive, I kept encountering an error stating `Verification failed: (0x1A) Security Violation`

![image](https://github.com/masoncooley/proxmox/assets/76832588/b999520a-ef93-404e-9727-89e128aa72a4)

I thought maybe this was a problem with Secure Boot, so I went into the UEFI/BIOS and disabled Secure Boot. I tried booting from the Ventoy drive once again and I received the same error. I went searching through the BIOS to find something that could cause this problem. I eventually gave up and looked up the error.

A quick Google search returned this Ubuntu forum which described my problem perfectly. https://askubuntu.com/questions/1456460/verification-failed-0x1a-security-violation-while-installing-ubuntu 

**Instructions from the forum:** 

`Press OK, Press any key to perform MOK management, Enroll key from disk, VTOYEFI, ENROLL_THIS_KEY_IN_MOKMANAGER.cer, Continue, Yes, Reboot.`

I followed the instructions and it booted right into the Proxmox installation GUI.

## Initial Configuration
After signing into the web GUI, I then opened Powershell and started an SSH session to the Proxmox server.

I then started following [this](https://youtu.be/GoZaMgEgrHw?si=gl4W-iVZFdUT4r8A) Techno Tim Youtube video.

### 1. Update Proxmox without subscription

We have to edit the sources list for updates by entering `nano /etc/apt/sources.list`, and adding two lines (one comment and one command.)
```
# not for production use
deb http://download.proxmox.com/debian buster pve-no-subscription
```

Since I don't have a Proxmox subscription, I'm going to go to the enterprise sources list with `nano /etc/apt/sources.list.d/pve-enterprise.list` and comment out the first line so it looks like this: `# deb https://enterprise.proxmox.com/debian/pve bookworm pve-enterprise`. This will prevent me from getting an error when I try to update.

Then I'll run `apt-get update` to update from the sources list I just edited. It gives me an error.

![image](https://github.com/masoncooley/proxmox/assets/76832588/70353296-db9e-4605-ab1a-01f1ee1c2bc0)

I noticed that all of the sources on my list use _bookworm_ whereas his list shows all of them having _buster_. I assumed that maybe the error was coming from the fact that I entered the new source using the _buster_ repository, so I changed it to _bookworm_, to no effect.

I went searching in the Proxmox PVE documentation and found the solution in **Topic 3.1.3. Proxmox VE No-Subscription Repository** which gives me the sources file contents for non-subscription users.

```
deb http://ftp.debian.org/debian bookworm main contrib
deb http://ftp.debian.org/debian bookworm-updates main contrib

# Proxmox VE pve-no-subscription repository provided by proxmox.com,
# NOT recommended for production use
deb http://download.proxmox.com/debian/pve bookworm pve-no-subscription

# security updates
deb http://security.debian.org/debian-security bookworm-security main contrib
```
I was still getting an error for `https://enterprise.proxmox.com/debian/ceph-quincy bookworm InRelease` which confused me because I didnt see that repository on either sources list. I scrolled down and saw **Topic 3.1.5. Ceph Reef Enterprise Repository** which told me that I dont need that repository active since I won't be using ceph. I then went into the sources file `/etc/apt/sources.list.d/ceph.list` and commented out that repo. This fixed all my errors.

### 2. Configuring storage

I want to host my OS on my SATA SSD and my VM storage on my NVME drive. I also want my VM storage to use the ZFS file system mainly because of the ability to make snapshots. When I try to make a ZFS disk using the web interface, there are no disks showing. So I'm going to use the `fdisk` command to remove all partitions from my NVME drive `/dev/nvme0n1`. I run the command `fdisk /dev/nvme0n1` to select the NVME drive. I then use `p` to view the existing partitions and `d` to delete the partition. If a disk has more than 1 partition it will usually prompt to ask which partition to delete but since my disk has only 1, there was no prompt. I use `p` again to make sure the disk is clean and `w` to save my changes.

I then want to make sure both of my drives have the SMART functionality enabled so that I can monitor them and ensure I have no data loss. To do this, I run `smartctl -a /dev/sda` for my boot drive and `smartctl -a /dev/nvme0n1` for my VM drive. Both drives passed the SMART tests.

I want a place to store my backups so I am going to connect an NFS share from my existing TrueNAS server. Using the web interface, I went to `Datacenter -> Storage -> NFS` and added the IP address of my NAS and the dataset my NFS share is mapped to. I got this error. 

![image](https://github.com/masoncooley/proxmox/assets/76832588/bd3b83e5-0ae8-4575-b1d2-6f0943587b23)

I found [this Reddit thread](https://www.reddit.com/r/Proxmox/comments/tz2ykj/comment/i427ga2/?utm_source=share&utm_medium=web3x&utm_name=web3xcss&utm_term=1&utm_content=share_button) that resolved my error. 

### 3. Scheduling backups

WIth my new NFS share, I want to schedule backups of my data with `Datacenter -> Backups -> Add`. 

![Screenshot 2024-06-05 213258](https://github.com/masoncooley/proxmox/assets/76832588/8baeab99-cbd8-41e9-a933-79bae6941c6b)

