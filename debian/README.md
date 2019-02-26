# Debian databox build

Alpine64 is failing to build driver-os-monitor. Don't know why.

## Add a second larger disk

Default VirtualBox/Vagrant debian disk is only 10GB - not big enough.

```
vagrant up
vagrant halt
```

In VirtualBox settings add a new disk to the SCSI device (second), say VDI, 50GB.

Start VM again. `vagrant up`

Partition disk (/dev/sdb), create ext4 filesystem, add to fstab mounted on /var.
```
sudo fdisk -l
sudo fdisk /dev/sdb
n
p
1


w
sudo mkfs.ext4 /dev/sdb1
sudo vi /etc/fstab
/dev/sdb1 /var  ext4
mount /dev/sdb1
```
Reboot.

## set up

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

## App

```
got clone https://github.com/cgreenhalgh/app-audit.git
cd app-audit
make build-amd64 DEFAULT_REG=databoxsystems VERSION=0.5.2
```
Open app store in Databox UI and add manifest.
