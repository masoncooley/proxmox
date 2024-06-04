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
