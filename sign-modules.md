Since kernel version 4.4.0-20, it was enforced that unsigned kernel modules will not be allowed to run with Secure Boot enabled. If you'd want to keep Secure Boot and also run these modules, then the next logical step is to sign those modules.

So let's try it.

Create signing keys

openssl req -new -x509 -newkey rsa:2048 -keyout MOK.priv -outform DER -out MOK.der -nodes -days 36500 -subj "/CN=Descriptive name/"
Sign the module

sudo /usr/src/linux-headers-$(uname -r)/scripts/sign-file sha256 ./MOK.priv ./MOK.der /path/to/module
Note 1: There can be multiple files to be signed for a single driver/module, so /path/to/module may need to be replaced with $(modinfo -n <modulename>), e.g. $(modinfo -n vboxdrv)

Note 2: sudo kmodsign sha512 ./MOK.priv ./MOK.der /path/to/module is an alternative if sign-file is not available.

Register the keys to Secure Boot

sudo mokutil --import MOK.der
Supply a password for later use after reboot

Reboot and follow instructions to Enroll MOK (Machine Owner Key). Here's a sample with pictures. The system will reboot one more time.

If the key has been enrolled properly, it will show up under sudo mokutil --list-enrolled.

Please let me know if your modules would run this way on Ubuntu 16.04 (on kernel 4.4.0-21, I believe).

Resources: Detailed website article for Fedora and Ubuntu implementation of module signing. (they've been working on it) ;-)

Additional resource: I created a bash script for my own use every time virtualbox-dkms upgrades and thus overwrites the signed modules. Check out my vboxsign originally on GitHub.

Additional note for the security (extra-)conscious: ;-)

Since the private key you created (MOK.priv in this example) can be used by anyone who can have access to it, it is good practice to keep it secure. You may chmod it, encrypt (gpg) it, or store it somewhere else safe(r). Or, as noted in this comment, remove the option -nodes in step number 1. This will encrypt the key with a passphrase.