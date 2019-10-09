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
If you get an error from the first `vagrant up` just try again...

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

## Sensing Kit

With 0.5.2...
Installed OK from phone.
Turning on light makes something happen in driver (including a memory leak error warning)
But light graph application didn't show anything or give any output.

Updated both to use lib-node-databox 0.5.2 and seems ok.

## changing default components

Note, even for apps/drivers that are in the app store (i.e. not just core components)
it always seems to use the dockerhub versions.

Try overiding the manifest store on databox start...
```
... --manifestStore https://github.com/cgreenhalgh/databox-manifest-store
```

### using custom components

Another option might be to use a custom version/tag, say `local`.

e.g. pull core components & tag
```
docker pull databoxsystems/databox:0.5.2
docker tag databoxsystems/databox:0.5.2 databoxsystems/databox:cmg
IMAGES="container-manager core-network core-network-relay core-store arbiter export-service core-ui driver-app-store driver-tplink-smart-plug driver-os-monitor app-os-monitor driver-sensingkit app-light-graph driver-twitter"
for i in $IMAGES; do 
  docker pull databoxsystems/$i-amd64:0.5.2
  docker tag databoxsystems/$i-amd64:0.5.2 databoxsystems/$i-amd64:cmg
done
```
or arm
```
for i in $IMAGES; do 
  docker pull databoxsystems/$i-arm64v8:0.5.2
  docker tag databoxsystems/$i-arm64v8:0.5.2 databoxsystems/$i-arm64v8:cmg
done
```

run using that tag
```
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock --network host -t databoxsystems/databox:cmg /databox start -sslHostName $(hostname) -release cmg
```

As of 2019-09-26, 0.5.2, I have fixes for 
- [driver-sensingkit cgreenhalgh branch lib-node-0.10](https://github.com/cgreenhalgh/driver-sensingkit/tree/lib-node-0.10)
- [app-light-graph cgreenhalgh branch lib-node-0.10](https://github.com/cgreenhalgh/app-light-graph/tree/lib-node-0.10)
- [databox-quickstart node app and driver, go app and driver arm build, cgreenhalgh branch 0.10-export](https://github.com/cgreenhalgh/databox-quickstart/tree/0.10-export)
- [driver-tplink-smart-plug cgreenhalgh branch fast-plug](https://github.com/cgreenhalgh/driver-tplink-smart-plug/tree/fast-plug)
- [driver-twitter cgreenhalgh branch no-user-stream](https://github.com/cgreenhalgh/driver-twitter/tree/no-user-stream)
- [driver-truelayer cgreenhalgh branch master](https://github.com/cgreenhalgh/driver-truelayer)
- [core-ui ktg branch master](https://github.com/ktg/core-ui)

and my extra components:
- [driver-message-view cgreenhalgh](https://github.com/cgreenhalgh/driver-message-view)
- [app-automate cgreenhalgh](https://github.com/cgreenhalgh/app-automate)
- [driver-button-view cgreenhalgh](https://github.com/cgreenhalgh/driver-button-view)
- [driver-mock-smart-plug](https://github.com/cgreenhalgh/driver-mock-smart-plug)

### re-tagging

So i'll re-tag some for use in class...
```
IMAGES="container-manager core-network core-network-relay core-store arbiter export-service core-ui driver-app-store driver-tplink-smart-plug driver-os-monitor app-os-monitor driver-sensingkit app-light-graph driver-twitter driver-truelayer driver-message-view driver-button-view app-automate"
for i in $IMAGES; do 
  docker tag databoxsystems/$i-amd64:cmg cgreenhalgh/$i-amd64:ena2019
done
for i in $IMAGES; do 
  docker push cgreenhalgh/$i-amd64:ena2019
done
```

Run my version...if dare...
```
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock --network host -t \
databoxsystems/databox:0.5.2 /databox start -sslHostName $(hostname) \
-registry cgreenhalgh -release ena2019 \
-cm cgreenhalgh/container-manager -core-ui cgreenhalgh/core-ui \
-arbiter cgreenhalgh/arbiter -core-network cgreenhalgh/core-network \
-core-network-relay cgreenhalgh/core-network-relay -app-server cgreenhalgh/driver-app-store \
-export-service cgreenhalgh/export-service -store cgreenhalgh/core-store \
-manifestStore https://github.com/cgreenhalgh/databox-manifest-store
```
(add `-flushSLAs` to avoid restarting old components automatically)

and stop...
```
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock --network host -t \
databoxsystems/databox:0.5.2 /databox stop
```

### to try...

1. install [docker](www.docker.com)
1. start databox (see above)
1. if planning to use the app, extract the QR code as above
1. (optional) install app on phone and scan the QR code to connect/log in
1. (and/or) open browser and direct to 127.0.0.1, follow link to "DATABOX DASHBOARD", persuading your browser to proceed in spite

1. Select App Store, then driver-os-monitor and install
1. Select App Store, then app-os-monitor; for each datasource (selector) select the (only) available option, then install
1. Select app-os-monitor - you should see some load and memory usage graphs

1. Select App Store, then driver-sensingkit and install
1. Select App Store, then app-light-graph, select input, then install
1. Select app-light-graph and look at the (empty) graph
1. In the databox phone app, select Sensing Kit, and tick the checkbox "Light"
1. With the app on/visible, put your hand over the light sensor and watch the app-light-graph change...

1. From app store, install: driver-button-view, driver-message-view, driver-twitter, driver-truelayer

1. If you have a TPlink smart plug then install driver-tplink-smart-plug, open the driver-tplink-smart-plug, enter the local subnet IP prefix and press scan. repeat until any local smart plugs are visible
1. If you don't then install driver-mock-smart-plug, open its UI and copy to a new tab (it will show the plug "state")

1. Open the driver-message-view; copy to a new tab
1. Open the driver-message-button; copy to a new tab
1. From app store select app-automate, select an input for every datasource and install
1. Open app-automate and play. e.g. cover the phone light sensor and the plug should go off and a message should be sent. Use the on/off "fire" options to turn the plug on/off.
1. Read the app-automate [docs](https://github.com/cgreenhalgh/app-automate) and add some rules - have fun...

1. (optional) From main menu, open driver-twitter and configure - see [instructions]
(https://github.com/cgreenhalgh/driver-twitter/tree/no-user-stream) - note that some of the oauth steps can only be done from a browser on 127.0.0.1
1. (optional) From main menu, open driver-truelayer and configure - see [instructions](https://github.com/cgreenhalgh/driver-truelayer) - note that some of the oauth steps can only be done from a browser on 127.0.0.1
1. go back to app-automate and do something with those data sources

## oddities and issues

### on arm...

I got errors from app-os-monitor (0.5.2 standard image)...
```
app-os-monitor.1.0ivmx5f2kyz7@databox    | tcp://driver-os-monitor-core-store:5555
app-os-monitor.1.0ivmx5f2kyz7@databox    | makeArbiterPostRequest /token Error::  Can't connect so server
app-os-monitor.1.0ivmx5f2kyz7@databox    | Error getting last N  freemem Error getting Arbiter Token: 500:
...
```
resolved by rebuilding after fixing Dockerfile-arm64v8 to specify alpine:3.8 
for deploy as well as build image base.

### connect to new component ui often fails initially

Just stalls then times out.
Usually ok on reload.

### resource temporarily unavailable

seem to be getting this sometimes. 
tends to be failure of get token from arbiter.
e.g. from core-ui to arbiter.
apparently this can be caused by a non-blocking socket write failing.

trying newer core-ui self-build... (no news yet)

### driver not starting properly

sometime the new driver-message-view stalls after registering one datasource,
but not the second, or getting old values.
is it a race with opening the ws to the client?? 

maybe driver-button-view doesn't always get the initial value either?!
