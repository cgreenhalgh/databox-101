# databox dev notes

Chris Greenhalgh, chris.greenhalgh@nottingham.ac.uk, 2017-09-25

Walking through getting started with Databox...

(and find on Windows 7 I need to run in a VM anyway - see below)

## Initial setup

See https://github.com/me-box/databox

Install docker for mac/windows (or docker toolbox if Windows < 10) (and docker-compose if not present)

```
git clone https://github.com/me-box/databox.git
```
Note, dev command-line option not in https://github.com/me-box/databox/blob/master/README.md as of 2017-09-25
```
cd databox
./databox-start dev
```

minor issue:
- install/start OS monitor driver and view UI quickly -> no reported error (UI stall, underlying 404)


How to access from mobile app?

### Docker in VM

(Docker Toolbox issues)

(Vagrant 2.0.0)
```
vagrant init ubuntu/xenial64
```
Increase memory to at least 2GB (1GB isn't enough)
```
vagrant up
```
See [docs](https://docs.docker.com/engine/installation/linux/docker-ce/ubuntu/#install-using-the-repository)
```
sudo apt-get update
sudo apt-get install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo apt-key fingerprint 0EBFCD88
```
Verify that you now have the key with the fingerprint 9DC8 5822 9FC7 DD38 854A E2D8 8D81 803C 0EBF CD88, 
```
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
sudo apt-get update
sudo apt-get install -y docker-ce
```
Optional,
```
sudo docker run hello-world
```
Docker compose, see [docs](https://docs.docker.com/compose/install/#install-compose):
```
sudo curl -L https://github.com/docker/compose/releases/download/1.16.1/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```
Optionally,
```
docker-compose --version
```

Note, in VM running docker requires sudo which is not used in the scripts...
See [docs](https://docs.docker.com/engine/installation/linux/linux-postinstall/#manage-docker-as-a-non-root-user)
```
sudo usermod -aG docker $USER
```
Log out / reboot VM

Run on boot...
```
sudo systemctl enable docker
```

Open port 8989 (main UI), 8181 (CM API?), 8086 (SDK), 9090 (SDK test).

### Docker toolbox issues

```
$ ./databox-start dev
```

#### Error 1
```
[2017-09-25T11:31:51+0100 databox-start]: docker is not installed;see https://docs.docker.com/engine/installation/
```
Fails to detect docker installed.
See https://github.com/me-box/databox/issues/98 

#### Error 2
```
...
[2017-09-25T11:41:53+0100 databox-start]: extracting host interface IP address ...
./databox-start: line 71: ifconfig: command not found
[2017-09-25T11:41:53+0100 databox-start]: host interface IP address =
```
External IP address determination fails with lack of ifconfig.

```
export EXT_IP=128.243.22.74
```

```
Error response from daemon: must specify a listening address because the address to advertise is not recognized as a system address, and a system's IP address to use could not be uniquely identified
```

### Docker on linux issues

#### docker-compose

make sure you install the latest one as per the docker website, don't just
apt-get install docker-compose!

#### networking / dns

Probably specific to local network configuration and/or blocking 8.8.8.8 nameserver.
Actually, probably only because I plugged it into the Ethernet and I probably have
a non-standard set-up

Initially visible as fail to buil code-container-manager at 3rd step (npm) with EAI_AGAIN i.e. DNS timeout.

Verified by 
```
docker run -it --rm node:alpine /bin/sh
ping www.google.com
```

for fix see [this answer](https://stackoverflow.com/questions/24991136/docker-build-could-not-resolve-archive-ubuntu-com-apt-get-fails-to-install-a)
which involves getting local organisation DNS servers and forcing docker to use them:
```
nmcli dev show | grep 'IP4.DNS'
sudo su root
cd /etc/docker
vi daemon.json
{
  "dns": ["X.X.X.X", "X.X.X.X"]
} 
exit
sudo service docker restart
```

## SDK

```
./databox-start sdk
```

Follow on-screen instructions (open http://127.0.0.1:8086/, go to github, set up oatch, copy ID/key to sdk/settings.json, restart)

Make app

test - ok?? don't see any messages for any of the built-in sources.
save - ok
publish - hangs on 'http://localhost:8086/github/publish' -> 500

Note: need to reload databox app to get new app in store view?!

Start - ok?!
View UI - cannot get ...

## Mobile app (test)

### Emulator

Install Android Studio. Standard start-up.

(using Android 7.1 google play image) doesn't find databox app in app store.

Installing directly from APK:
- cannot discover databox
- host IP => access OK
- can start sensing but driver cannot connect back to phone 
  - probably the emulator is behind its own NAT (reports using a 10.... address)
  - sensingkit runs nanohttpd on port 8080 on phone (?!)

Tried [port forwarding](https://developer.android.com/studio/run/emulator-networking.html)
to emulator but connections are immediately terminated. 

### Physical device

#### Eduroam 

no access to my desktop

#### UoN VPN

[VPN info](http://www.nottingham.ac.uk/it-services/connect/working/off-campus.aspx)

Install app, join VPN -> no access to my desktop.

### WiFi 

OK on my home network to my laptop (both on wireless).

WiFi adapter ... ?

### Laptop

Install ubuntu 16.04, docker, etc.

Local wired network issue:
node:alpine container can't resolve DNS addresses...!!
(OK on desktop, OK in terminal outside docker).

## VPN

### UoN VPN

UoN https://vpntest.nottingham.ac.uk

Groups:
- IPSEC_PRIVATE
- IPSEC_PUBLIC
- SSL_PRIVATE
- SSL_PUBLIC

Configure using openconnect, [info](https://github.com/dnschneid/crouton/wiki/Using-Cisco-AnyConnect-VPN-with-openconnect)

```
sudo apt-get install -y openvpn openconnect

VPNGRP=my-vpn-group
VPNUSER=my-vpn-user
VPNURL=https://vpn.mydomain.com

sudo openvpn --mktun --dev tun1 && \
sudo ifconfig tun1 up && \
sudo /usr/sbin/openconnect $VPNURL --user=$VPNUSER --authgroup=$VPNGRP --interface=tun1
i.e.
sudo /usr/sbin/openconnect https://vpntest.nottingham.ac.uk --user=pszcmg --authgroup=IPSEC_PRIVATE --interface=tun1

sudo ifconfig tun1 down
sudo openvpn --rmtun --dev tun1

```

With IPSEC_PRIVATE remote server sees client IP:
- Browser on host: 128.243.22.74
- VM w Tunnel up: 10.159.250.18
- VM w tunnel down: 128.243.22.74

But two devices on VPN can't communicate (on port 8989, anyway!)
Perhaps just local filtering/firewall rules?

Cannot connect IPSEC_PUBLIC.

From UoN server can ping 10.... internal IP address.

### Own VPN

... ??

## Build an app

