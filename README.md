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
## Databox use

Start databox as per [main docs](https://github.com/me-box/databox)

You can log in on the same machine - always runs on default HTTP/HTTPS ports (80/443).
And/or if using Vagrant NAT then set up port forwarding on that interface. 
But note that forwarding non-default ports probably won't work (e.g. for some drivers, 
oauth, etc.).

The password is printed on the databox logs/console on start-up - 
it's quite long.

There is a QR Code that can be used by the app to join (with host and password).
It is supposed to be visible on the settings page once logged in, but doesn't seem to
be (at all on localhost, broken on mobile app).

The app QR code can be downloaded using curl (etc),
first send the password as an authorization token to get a session cookie,
```
curl -X GET -H 'Authorization: Token PASSWORD' --insecure -c databoxcookies https://127.0.0.1/
curl -o qrcode.png -X GET -b databoxcookies --insecure https://127.0.0.1/core-ui/ui/api/qrcode.png
```

You can then scan it on the app to access the databox.

### SDK use

Start databox SDK as per [main docs](https://github.com/me-box/databox)

Map and use [port 8086](http://127.0.0.1:8086)

Fails on start-up:
```
[INFO]2019/09/04 09:05:57 Pulling Image tlodge/databox-sdk
[INFO]2019/09/04 09:06:13 Done pulling Image tlodge/databox-sdk
...
MongoError: failed to connect to server [localhost:27017] on first connect [MongoError: connect ECONNREFUSED 127.0.0.1:27017]
```

## Databox bugs

The QR Code is supposed to be visible on the settings page once logged in, but doesn't seem to
be (at all on localhost, broken on mobile app).

I can't change the Databox name (in core-ui Settings).

Logs of settings operations appear unimplemented, e.g. Delete data.
So after uninstalling and re-installing an app the data (inc settings) are still there.

When a driver/app is starting up you quite often see error responses when attempting
to access its UI (`connection refused` when the driver's embedded web server isn't fully started).

App-os-monitor says "Stats for last 5 minuets [sic]" :-)

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

## TrueLayer Driver

The [TrueLayer driver]() downloads (polls) an account - balance and transactions.

Set up only works from the local browser (127.0.0.1) as the redirect URLs
are hard-coded to 127.0.0.1.

The driver doesn't handle e.g. cancelling the authorization 
(redirect with parameters `error=access_denied`).

The driver doesn't ask for the testing bank.

When successful the final redirect is direct to the driver ui in a separate 
tab, and doesn't redirect into the main app or close.

The driver worked for me with my Lloyds account (Minos has been testing with
Monzo).
