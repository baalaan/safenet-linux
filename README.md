# safenet-linux

## Introduction

How to configure SafeNet eToken in Linux

### Prerequisites

* SafeNet Authentication Client >= 10.9 installed
  - Download: https://knowledge.digicert.com/general-information/how-to-download-safenet-authentication-client
* libnss3 and modutil (libnss3-tools)
  - Ubuntu/Linux Mint: `sudo apt install libnss3 libnss3-tools`
  - Arch Linux: `sudo pacman -S nss`

### Tested On

* Linux Mint 22.3 (Firefox 134, Chromium 133, SAC 10.9)
* Arch Linux

## Step 1: Find the eToken Library Path

This step is required for both Firefox and Chromium/Chrome configuration.

Once SafeNet Authentication Client is installed, find the path of the eToken library:

```bash
$ find /lib* /usr/lib* /usr/local/lib* -name "*libeToken.so*"
/usr/lib/libeToken.so.10.7.77
/usr/lib/libeToken.so.10
/usr/lib/libeToken.so
```

Use the versioned `.so` file (e.g. `/usr/lib/libeToken.so.10`) rather than the unversioned symlink, as it is more stable across updates. The examples below use `/usr/lib/libeToken.so.10`.

## How to Configure Firefox

Open Firefox and go to the security preferences (`about:preferences#privacy` in the Firefox navbar).

At the end of the page, click on `Security Devices` and a new window will appear:

![security devices](images/security-devices.png)

Click on Load and enter the following information:

* Module name: `eToken PKCS#11 module`
* Module Filename: `/usr/lib/libeToken.so.10`

Click Ok.

Restart Firefox for the changes to take effect.

### Verify

After restarting Firefox, go back to `Security Devices`. Your eToken should appear in the list of security modules. You can also visit a site that requires your certificate to confirm the token is recognized.

## How to Configure Chromium / Google Chrome

Chromium and Google Chrome do not offer a graphical interface to manage PKCS#11 devices. Use `modutil` from libnss3-tools to configure it.

Both browsers share the same NSS database at `~/.pki/nssdb/`.

```bash
cd ~
modutil -dbdir sql:.pki/nssdb/ -add "eToken" -libfile /usr/lib/libeToken.so.10
# You will be prompted to confirm — press Enter to continue
modutil -dbdir sql:.pki/nssdb/ -list  # Verify that eToken appears in the list
```

If you see the error `NSS database not found` or `ERROR: NO DB FOUND`, create the database first:

```bash
mkdir -p ~/.pki/nssdb
modutil -dbdir sql:.pki/nssdb/ -create
```

Then re-run the `modutil -add` command above.

Restart your browser for the changes to take effect.

### Verify

After restarting, navigate to a site that requires your certificate. The browser should prompt you for your eToken PIN.
