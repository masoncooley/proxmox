# proxmox
## Detailing my new Proxmox server and documenting all the issues I run into.

### Installation
I first grabbed the Proxmox Virtual Environment 8.2.2 `.iso` from the official website and loaded it onto a Ventoy USB drive. While trying to boot from the installation drive, I kept encountering an error stating `Verification failed: (0x1A) Security Violation`

![image](https://github.com/masoncooley/proxmox/assets/76832588/b999520a-ef93-404e-9727-89e128aa72a4)

I thought maybe this was a problem with Secure Boot, so I went into the UEFI/BIOS and disabled Secure Boot. I tried booting from the Ventoy drive once again and I received the same error. I went searching through the BIOS to find something that could cause this problem. I eventually gave up and looked up the error.

A quick Google search returned this Ubuntu forum which described my problem perfectly. https://askubuntu.com/questions/1456460/verification-failed-0x1a-security-violation-while-installing-ubuntu 

**Instructions from the forum:** 

`Press OK, Press any key to perform MOK management, Enroll key from disk, VTOYEFI, ENROLL_THIS_KEY_IN_MOKMANAGER.cer, Continue, Yes, Reboot.`

I followed the instructions and it booted right into the Proxmox installation GUI.

### Initial Configuration
After signing into the web GUI, I then opened Powershell and started an SSH session to the Proxmox server.

I then started following [this](https://youtu.be/GoZaMgEgrHw?si=gl4W-iVZFdUT4r8A) Techno Tim Youtube video.

1. **Update Proxmox without subscription**

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
