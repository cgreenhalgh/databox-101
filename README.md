# Databox 101

Building/running, and basis for provenance explorations.

Databox 0.5.2, 2019-09-03

Chris Greenhalgh, The University of Nottingham

## Vagrant

Hitting obscure problems using [Alpine](alpine/README.md), 
so I suggest [Debian](debian/README.md) for now. 

Note, Docker & Databox are really fussy about networking and DNS config.
So best to leave DNS to default, but this may mean restarting VM/docker/Databox
when the network changes. 

Check /etc/resolv.conf for DNS config (picked up by docker) - 
remember UoN won't allow use of 8.8.8.8 from at least some of its networks.

Check that databox-network can (a) do arbitrary DNS lookups and 
(b) actually exchange traffic with external machines.

Also e.g. check that the app-store shows a list of apps - if not 
https from github is probably failing. Check databox-network for any errors
or refusal to look up. Check manual (wget) http from app-store container.

Note - sensitive to network interfaces - debian vagrant file has fixes to
make eth0 a brigded interface, which seems to work.

### Git setup

Consider setting username & email if committing with git:
```
git config --global user.email "XXX"
git config --global user.name "XXX"
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

