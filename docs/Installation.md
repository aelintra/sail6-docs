# Installation

V6 can be installed onto X86 or ARM platforms. Only 64-bit architectures are supported. It will happily install on bare-metal or in virtual environments and has additional support for AWS EC2 instances. SARK is developed on Ubuntu and it should ideally be installed onto that distro but you may be able to install it onto other "debian-like" distros (YMMV).

- It will work with most modern browsers.
- It will NOT work on any browser that doesn't support jQuery or has it disabled.
- It will NOT work on any browser that does not support HTML5 or CSS3.

## Before you begin

- V6.2 is designed to run as an appliance and must not be installed onto a multi-purpose system that has other software already installed on it. If you try, you will likely break both the existing systems and SARK. You must begin with an Ubuntu 22.04 minimal install which contains nothing but an SSH server.

## Installing the package manager

SARK uses the packagecloud repo.   Here are the steps you need to prepare to use it.

```sh
sudo apt update
sudo apt upgrade
curl -s https://packagecloud.io/install/repositories/aelintra/SARK/script.deb.sh | sudo bash
```


## SARK 6.2 Installation

The latest SARK 6.2 will auto-install on Ubuntu 22.04 LTS (Jammy Jellyfish).  It may install on other debian-like releases, but you will need to grab the deb packages from GitHub and install them manually (https://github.com/aelintra/sail6).


### Installing a Mail Agent

If this is a new install, you should install a mail agent. SARK has on-board support for a lightweight mail agent called ssmtp. If you install it, then SARK will provide a tab in the networking section for you to configure it. Install it now (so that SARK can set the correct permissions) ...:

```sh
sudo apt install ssmtp
```

**Note:** Ubuntu creates the ssmtp directory with secure access permissions. You must fix it by doing:

```sh
sudo chmod +x /etc/ssmtp
```

### Install mySQL/Mariadb

The SARK install uses an SQL database to process call statistics.  It can be either mysql or mariadb but SARK does not specify which, so you'll need to decide. Install it now ...:  

```sh
sudo apt install mysql-server
```

OR

```sh
sudo apt install mariadb-server
```
**N.B. DO NOT run mysql_secure_installation yet!!**   The SARK install will automatically build the CDR Database for you but it needs access to mysql root without a password.   Once the installation is done, you *should* then run **mysql_secure_installation** to secure your DB instance (see below).

### Install the SARK packages

```sh
sudo apt install sail
```

The install will take a good few minutes depending upon the speed of the donor box and your internet link. During the install, you may be asked to enter an admin password for LDAP (make a note of it, you'll need it later). It will also ask about dumpcap but just take the default (No) and let the install run to its conclusion.

### link the HPE according to your architecture
SARK ships with a compiled C code switching engine called sarkhpe.  There are two versions; one for regular X86-64 type architectures and one for the increasingly popular ARM64.

If you don't know which arch you are running do this...

````sh
uname -m
````

It will return something like *X86-64* (Intel/AMD) or *aarch64* (ARM).  Once you have decided which architecture you have, proceed as follows..

```sh
cd /usr/share/asterisk/agi-bin
ls -l
-rwxr-xr-x 1 root root  19525 Dec 14  2023 kwakeup
-rwxr-xr-x 1 root root 530944 Mar 12 19:00 sarkhpe_amd64
-rwxr-xr-x 1 root root 545528 Mar 12 19:00 sarkhpe_arm64
```

As you can see, the two HPE modules each have a suffix denoting the arch for which they were compiled.
Choose the one you want by creating a soft link to it...

```sh
sudo ln -s sarkhpe_amd64 sarkhpe
```

Now the file list should show the link you created
 this...

```sh
ls -l 
-rwxr-xr-x 1 root root  19525 Dec 14  2023 kwakeup
lrwxrwxrwx 1 root root     13 Mar 13 18:47 sarkhpe -> sarkhpe_amd64
-rwxr-xr-x 1 root root 530944 Mar 12 19:00 sarkhpe_amd64
-rwxr-xr-x 1 root root 545528 Mar 12 19:00 sarkhpe_arm64
```

### Reboot

Now reboot your new PBX
```sh
sudo reboot
```

### Install Asterisk Extra Sounds Package

SARK requires the Asterisk extra sounds package. If you want UK English, there is a deb on the repo:

```sh
sudo apt install ast-en-gb-gpl-gsm-sounds
```

If you want "Alison" (US American), there is no deb available for this but it's pretty easy to install. At the Linux CLI, do the following:

```sh
cd /usr/share/asterisk/sounds
sudo wget http://downloads.asterisk.org/pub/telephony/sounds/asterisk-extra-sounds-en-gsm-current.tar.gz
sudo tar xvfz asterisk-extra-sounds-en-gsm-current.tar.gz
sudo rm asterisk-extra-sounds-en-gsm-current.tar.gz
```

### Secure your mysql/mariadb database

Now you can run the secure script for your mysql or maria database

```sh
sudo mysql_secure_installation
```
Follow the script prompts.


##Next steps

If you were running a previous version of SARK on your browser then you will need to clear your broswer cache or open an incognito/private window before proceeding. If you don't, you'll get odd-looking output as the cached jQuery code fights with the new V6 output.

Your SARK app will be at `https://your.server.ip.address`

Default login credentials will be...

- **UID:** admin
- **PWD:** sarkadmin

If this is a brand new install, the SARK browser application will force you to change your password on first login. This will change BOTH the browser AND linux root passwords.
