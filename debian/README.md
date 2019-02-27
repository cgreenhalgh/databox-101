# Debian databox build

Alpine64 is failing to build driver-os-monitor. Don't know why.

## Vagrant

If you're using Vagrant to make the debian VM...

### Add a second larger disk

Default VirtualBox/Vagrant debian disk is only 10GB - not big enough.

```
vagrant up
vagrant halt
```

In VirtualBox settings add a new disk to the SCSI device (second), say VDI, 50GB.

Start VM again. `vagrant up`

Partition disk (/dev/sdb), create ext4 filesystem, init, add to fstab mounted on /var.
```
sudo fdisk -l
sudo fdisk /dev/sdb
n
p
1


w
sudo mkfs.ext4 /dev/sdb1
sudo mkdir /mnt/tmp
sudo mount /dev/sdb1 /mnt/tmp
sudo mv /var/* /mnt/tmp
sudo vi /etc/fstab
/dev/sdb1 /var  ext4
```
Reboot.

### fix DNS

Note: consider DNS fix for vagrant: DNS can be more reliable if, once after first creating VM, you shut down VM (vagrant halt) and
```
vboxmanage list vms
vboxmanage modifyvm "debian_default_XXXX" --natdnshostresolver1 on
vagrant up
```
(on windows vboxmanage is in c:\Windows\Program Files\Oracle\VirtualBox)

### File sync

Note, once vagrant up, to keep syncing the /vagrant directory you need to (also) run
```
vagrant rsync-auto
```

### set up

Standard docker on debian install:
```
sudo apt-get update
sudo apt-get install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg2 \
    software-properties-common
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -
sudo apt-key fingerprint 0EBFCD88
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/debian \
   $(lsb_release -cs) \
   stable"
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io
sudo usermod -aG docker $USER
```
Log out; log back in if you want to test:
```
docker run hello-world
```
Add extra databox dependencies:
```
sudo apt-get install -y git wget 
sudo apt-get install -y build-essential 
curl -sL https://deb.nodesource.com/setup_10.x | sudo bash -
sudo apt-get install -y nodejs
```

## Databox

0.5.2 just released, so try local build from master...
```
git clone https://github.com/me-box/databox.git
cd databox
make build-linux-amd64 get-containers-src build-core-containers ARCH=amd64

make start ARCH=amd64

make build-app-drivers ARCH=amd64
```

NB need to use port 443 for SSH even on VM.

## App

```
got clone https://github.com/cgreenhalgh/app-audit.git
cd app-audit
make build-amd64 DEFAULT_REG=databoxsystems VERSION=0.5.2
```
Open app store in Databox UI and add manifest.

## databox issues

Attempt to upload my manifest to app-store settings returns "Incorrect file format" from Chrome/Windows.

When doing a local build and tag of (e.g.) databoxsystems/driver-app-store-amd64:0.5.2 starting databox still picks up the one from dockerhub.
E.g.
```
make build-amd64 DEFAULT_REG=cgreenhalgh VERSION=0.5.2
...
make start ARCH=amd64 OPTS="--app-server cgreenhalgh/driver-app-store"
```

Over-riding a single core component image (e.g. --app-store XXXX) at databox start doesn't let you use a different version tag, so you have to use a custom registry name to be sure of picking up the right image.
