# Databox 101

Building/running, and basis for provenance explorations.

Databox 0.5.2-dev, 2019-02-22

Chris Greenhalgh, The University of Nottingham

## Vagrant

Includes Alpine 64bit [vagrant](https://www.vagrantup.com/) file for (e.g.) Windows dev.

if using Vagrant
```
vagrant up
vagrant ssh
cd /vagrant
```

Note: consider DNS fix for vagrant: DNS can be more reliable if, once after first creating VM, you shut down VM (vagrant halt) and
```
vboxmanage list vms
vboxmanage modifyvm "databox-101_default_XXXX" --natdnshostresolver1 on
vagrant up
```

## Alpine set-up

```
sudo apk update
sudo apk upgrade
```

### Docker 

Install docker (to run without sudo):
```
sudo apk add docker
sudo rc-update add docker boot
sudo service docker start
sudo docker version
sudo adduser $USER docker
docker run hello-world
```
(version 18.09.1-ce)

Docker is installed in boot runlevel. 
By default docker needs sysfs, cgroups (only)

Docker files are all (by default) in /var/lib/docker (logs in /var/log/docker*)

non-root docker:
```
```


### Other Databox dependencies

(as of 0.5.2)
- git
- make
- wget
- npm/node (should go away as a dependency from 0.5.2 release)

```
sudo apk add git
sudo apk add build-base
sudo apk add wget
sudo apk add npm
```



### Git setup

Consider setting username & email if committing with git:
```
git config --global user.email "XXX"
git config --global user.name "XXX"
```

## Databox set-up

See [databox repo](https://github.com/me-box/databox).
Right now I'm working from branch 0.5.2-dev since it hasn't been released yet.

```
git clone https://github.com/me-box/databox.git
git fetch
git checkout 0.5.2-dev
```
Note, consider changing core-ui to use branch `0.5.2-dev` rather than `master` (in Makefile)

Local build...
```
make build-linux-amd64 get-containers-src build-core-containers ARCH=amd64
```

Run
```
make start ARCH=amd64
```

Stop 
```
make stop ARCH=amd64
```


Apps/drivers
```
make build-app-drivers ARCH=amd64
```

## Databox app/driver

See the [databox quickstart](https://github.com/me-box/databox-quickstart/).

It includes sample core code, e.g. [go app](https://github.com/me-box/databox-quickstart/tree/0.5.2-dev/go/app)

### Build

Clone into databox/build

```
make build-amd64 DEFAULT_REG=databoxsystems VERSION=0.5.2
```
Note, building the core-ui may not work over a mounted windows filesystem.

Open app store in Databox UI and add manifest.

