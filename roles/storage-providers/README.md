<p align="center"><img src="img/storage-provider_new.svg"></p>

<div align="center">
  <h4>This is a guide to setting up your <a href="https://github.com/Joystream/joystream/tree/master/storage-node">storage node</a>, and getting started as a Storage Provider on the latest
    <a href="https://testnet.joystream.org/">testnet</a>.</h4><br>
</div>



Table of Contents
==
<!-- TOC START min:1 max:3 link:true asterisk:false update:true -->
- [Overview](#overview)
- [Instructions](#instructions)
  - [Initial setup](#initial-setup)
  - [Install IPFS](#install-ipfs)
    - [Configure IPFS](#configure-ipfs)
    - [Run IPFS as a service](#run-ipfs-as-a-service)
    - [Update IPFS](#update-ipfs)
  - [Setup Hosting](#setup-hosting)
    - [Instructions](#instructions-1)
    - [Run caddy as a service](#run-caddy-as-a-service)
  - [Install and Setup the Storage Node](#install-and-setup-the-storage-node)
    - [Applying for a Storage Provider opening](#applying-for-a-storage-provider-opening)
    - [Setup and configure the storage node](#setup-and-configure-the-storage-node)
    - [Check that you are syncing](#check-that-you-are-syncing)
    - [Set an Empty Storage URL](#set-an-empty-storage-url)
    - [Run storage node as a service](#run-storage-node-as-a-service)
    - [Enable the Public URL When Synced](#enable-the-public-url-when-synced)
    - [Verify everything is working](#verify-everything-is-working)
  - [Update Your Storage Node](#update-your-storage-node)
    - [Update Colossus](#update-colossus)
  - [Update Older Storage Nodes](#update-older-storage-nodes)
    - [Update IPFS and Caddy](#update-ipfs-and-caddy)
    - [Update Colossus](#update-colossus-1)
    - [Start Up](#start-up-1)
- [Troubleshooting](#troubleshooting)
  - [Port not set](#port-not-set)
  - [No tokens in role account](#no-tokens-in-role-account)
  - [Caddy v1 (deprecated)](#caddy-v1-deprecated)
    - [Run caddy as a service](#run-caddy-as-a-service-1)
<!-- TOC END -->



# Overview

This page contains all information required to set up your storage node and become a `Storage Provider` on the current Joystream testnet.

The guide for the `Storage Provider Lead` can be found [here](/roles/storage-lead).

# Instructions

The instructions below will assume you are running as `root`. This makes the instructions somewhat easier, but less safe and robust.

Note that this has been tested on a fresh images of `Ubuntu 20.04 LTS`. You may run into some troubles with `Debian`.

The system has shown to be quite resource intensive, so you should choose a VPS with specs AT LEAST equivalent to [Linode 8GB](https://www.linode.com/pricing/) or better (not an affiliate link). Note that you unless you go for a very expensive out of the box node, have to add extra storage separately, and configure IPFS to use that disk.

Please note that unless there are any openings for new storage providers (which you can check in [Pioneer](https://testnet.joystream.org/) under `Working Groups` -> `Opportunities`), you will not be able to join. Applying to the opening is easiest in Pioneer, but once hired, you no longer need it. Actions you may want to perform after getting hired are easiest to carry out with the [CLI](/tools/cli/README.md#working-groups). With this, you can configure things like:
- changing your reward destination address
- changing your role key
- increasing your stake
- leaving the role

## Initial setup
First of all, you need to connect to a fully synced [Joystream full node](https://github.com/Joystream/joystream/releases). By default, the program assumes you are running a node on the same device. For instructions on how to set this up, go [here](/roles/validators). Note that you can disregard all the parts about keys before applying, and just install the software so it is ready to go.
We strongly encourage that you run both the [node](/roles/validators#run-as-a-service) and the other software below as a service.

Now, get the additional dependencies:
```
$ apt-get update && apt-get upgrade -y
# on debian 10, if you manage the first hurdle:
$ apt-get install libcap2-bin
```

## Install IPFS
The storage node uses [IPFS](https://ipfs.io/) as backend.
```
$ wget https://github.com/ipfs/go-ipfs/releases/download/v0.9.1/go-ipfs_v0.9.1_linux-amd64.tar.gz
$ tar -xvzf go-ipfs_v0.9.1_linux-amd64.tar.gz
$ cd go-ipfs
$ ./ipfs init --profile server
$ ./install.sh
# start ipfs daemon:
$ ipfs daemon
```
If you see `Daemon is ready` at the end, you are good!

### Configure IPFS
Some of the default configurations needs to be changed, in order to get better performance:

```
# cuz xyz
$ ipfs config --bool Swarm.DisableBandwidthMetrics true

# Default only allows storing 10GB. At the time of writing, ~650GB of disk space is needed, but this will likely grow fast, and you might forget so:
$ ipfs config Datastore.StorageMax "2000GB"

# cuz xyz
$ ipfs config --json Gateway.PublicGateways '{"localhost": null }'

# as outline here: https://github.com/lucas-clemente/quic-go/wiki/UDP-Receive-Buffer-Size
$ sysctl -w net.core.rmem_max=2500000

# cuz xyz
$ ipfs config --bool Gateway.NoFetch true
```


### Run IPFS as a service

To ensure high uptime, it's best to set the system up as a `service`.

Example file below:

```
$ nano /etc/systemd/system/ipfs.service
# Paste in everything below the stapled line
---
[Unit]
Description=ipfs
After=network.target

[Service]
Type=simple
User=root
WorkingDirectory=/root
LimitNOFILE=10240
PIDFile=/run/ipfs/ipfs.pid
ExecStart=/usr/local/bin/ipfs daemon --routing=dhtclient
Restart=on-failure
RestartSec=3
StartLimitInterval=600

[Install]
WantedBy=multi-user.target
```
Save and exit. Close the `ipfs daemon` if it's still running, then:
```
$ systemctl start ipfs
# If everything works, you should get an output. Verify with:
$ systemctl status ipfs
# If you see something else than "Daemon is ready" at the end, try again in a couple of seconds.
# To have ipfs start automatically at reboot:
$ systemctl enable ipfs
# If you want to stop ipfs, either to edit the file or some other reason:
$ systemctl stop ipfs
```

### Update IPFS
If you installed using the instructions listed here, the easiest way to update IPFS is the use `ipfs-update`:

```
# Stop IPFS:
$ systemctl stop ipfs

# Then install the new version
$ wget https://dist.ipfs.io/ipfs-update/v1.7.1/ipfs-update_v1.7.1_linux-amd64.tar.gz
$ tar -vxf ipfs-update_v1.7.1_linux-amd64.tar.gz
$ cd ipfs-update
$ ./install.sh
$ ipfs-update install 0.9.1

# verify it worked
$ ipfs version
# which should return
ipfs version 0.9.1
```

Note that regardless of how you choose to update, if you have had IPFS installed for some time, and have been using the example service file we have provided, you may want to check that it says:

```
PIDFile=/run/ipfs/ipfs.pid
LimitNOFILE=10240


# NOT
PIDFile=/var/run/ipfs/ipfs.pid
LimitNOFILE=8192
```

Finally, make sure you have set all the items listed under [Configure IPFS](#configure-ipfs) - some of them are rather new, and some are updated!


## Setup Hosting
In order to allow for users to upload and download, you have to setup hosting, with an actual domain as both Chrome and Firefox requires `https://`. If you have a "spare" domain or subdomain you don't mind using for this purpose, go to your domain registrar and point your domain to the IP you want. If you don't, you will need to purchase one.

To configure SSL-certificates the easiest is to use [caddy](https://caddyserver.com/), but feel free to take a different approach. Note that if you are using caddy for commercial use, you need to acquire a license. Please check their terms and make sure you comply with what is considered personal use.

Previously, this guide was using Caddy v1, but this has now been deprecated. As some of you may already have installed it, and may want to continue running it, the now deprecated instructions can be found in full [here](#caddy-v1).

### Instructions
For the best setup, you should use the "official" [documentation](https://caddyserver.com/docs/).

The instructions below are for Caddy v2.4.1:
```
$ wget https://github.com/caddyserver/caddy/releases/download/v2.4.1/caddy_2.4.1_linux_amd64.tar.gz
$ tar -vxf caddy_2.4.1_linux_amd64.tar.gz
$ mv caddy /usr/bin/
# Test that it's working:
$ caddy version
```

Configure the `Caddyfile`:
```
$ nano ~/Caddyfile
# Paste in everything below the stapled line
---
# Storage Node API
https://<your.cool.url/storage>/* {
        route /storage/* {
                uri strip_prefix /storage
                reverse_proxy localhost:3000
        }
        header /storage {
                Access-Control-Allow-Methods "GET, PUT, HEAD, OPTIONS"
        }
        request_body {
                max_size 20GB
        }
}
```

Now you can check if you configured correctly, with:
```
$ caddy validate ~/Caddyfile
# Which should return:
--
...
Valid configuration
--
# You can now run caddy with:
$ caddy run --config /root/Caddyfile
# Which should return something like:
--
...
... [INFO] [<your.cool.url>] The server validated our request
... [INFO] [<your.cool.url>] acme: Validations succeeded; requesting certificates
... [INFO] [<your.cool.url>] Server responded with a certificate.
... [INFO][<your.cool.url>] Certificate obtained successfully
... [INFO][<your.cool.url>] Obtain: Releasing lock
```

### Run caddy as a service
To ensure high uptime, it's best to set the system up as a `service`.

Example file below:

```
$ nano /etc/systemd/system/caddy.service
# Paste in everything below the stapled line
---
[Unit]
Description=Caddy
Documentation=https://caddyserver.com/docs/
After=network.target

[Service]
User=root
ExecStart=/usr/bin/caddy run --config /root/Caddyfile
ExecReload=/usr/bin/caddy reload --config /root/Caddyfile
TimeoutStopSec=5s
LimitNOFILE=1048576
LimitNPROC=512
PrivateTmp=true
ProtectSystem=full
AmbientCapabilities=CAP_NET_BIND_SERVICE

[Install]
WantedBy=multi-user.target
```
Save and exit. Close `caddy` if it's still running, then:
```
$ systemctl start caddy
# If everything works, you should get an output. Verify with:
$ systemctl status caddy
# Which should produce something similar to the previous output.
# To have caddy start automatically at reboot:
$ systemctl enable caddy
# If you want to stop caddy:
$ systemctl stop caddy
# If you want to edit your Caddyfile, edit it, then run:
$ caddy reload
```

## Install and Setup the Storage Node

First, you need to clone the Joystream monorepo, which contains the storage software.
Note that if you already have a storage-node installed (or running), go [here](#update-your-storage-node).

```
$ git clone https://github.com/Joystream/joystream.git
$ cd joystream
$ ./setup.sh
# this requires you to start a new session. if you are using a vps:
$ exit
$ ssh user@ipOrURL
# on your local machine, just close the terminal and open a new one
$ yarn build:packages
$ yarn run colossus --help
```

### Applying for a Storage Provider opening

Click [here](https://testnet.joystream.org) to open the `Pioneer app` in your browser. Then follow instructions [here](https://github.com/Joystream/helpdesk#get-started) to generate a set of `Keys`, get tokens, and sign up for a `Membership`. This `key` will be referred to as the `member` key.

Make sure to save the `5YourJoyMemberAddress.json` file. This key will require tokens to be used as stake for the `Storage Provider` application (`application stake`) and further stake may be required if you are selected for the role (`role stake`).

To check for current openings, visit [this page](https://testnet.joystream.org/#/working-groups/opportunities) on Pioneer and look for any `Storage Provider` applications which are open for applications. If there is an opening available, fill in the details requested in the form required and stake the tokens needed to apply (when prompted you can sign a transaction for this purpose).

During this process you will be provided with a role key, which will be made available to download in the format `5YourStorageAddress.json`. If you set a password for this key, remember it :)

The next steps (below) will only apply if you are a successful applicant.


### Setup and configure the storage node

**Make sure your [Joystream full node](https://github.com/Joystream/joystream/releases) is fully synced before you move to the next step(s)!**

Assuming you are running the storage node on a VPS via ssh, on your local machine:

```
# Go the directory where you saved your <5YourStorageAddress.json>:
$ scp <5YourStorageAddress.json> <user>@<your.vps.ip.address>:/root/joystream/storage-node/
```
Your `5YourStorageAddress.json` should now be where you want it.

On the machine/VPS you want to run your storage node:

```
# If you are not already in that directory:
$ cd ~/joystream/storage-node
```

On our older testnets, at this point you would have to "apply" using a separate colossus command to any available storage role. With the evolution of our testnet and the introduction of the `Storage Working Group`, this is no longer necessary. The next steps simply require that you link the "role key" (`5YourStorageAddress.json`) and `Storage ID` to your storage server.

To check your `Storage ID`, you have two (easy) options:
1. Use the [CLI](/tools/cli/README.md#working-groups:overview)
2. Check [Pioneer](https://testnet.joystream.org/#/working-groups)

**Note:** Make sure you send some tokens to your "role key"/`5YourStorageAddress.json` before proceeding! It needs tokens to send transactions, or it will be considered "down", and unavailable for syncing.

```
# To make sure everything is running smoothly, it would be helpful to run with DEBUG.

$ cd ~/joystream
$ DEBUG=joystream:* yarn run colossus server --key-file <5YourStorageAddress.json> --public-url https://<your.cool.url>/storage/ --provider-id <your_storage-id>

# If you set a passphrase for <5YourStorageAddress.json>:
$ DEBUG=joystream:* (yarn run) colossus server --key-file <5YourStorageAddress.json> --public-url https://<your.cool.url>/storage/ --provider-id <your_storage-id> --passphrase <your_passphrase>
```

If you do this, you should see (among other things) something like:

```
...  ________                     _____
...  ______(_)__________  __________  /__________________ _______ ___
...  _____  /_  __ \_  / / /_  ___/  __/_  ___/  _ \  __ `/_  __ `__ \
...  ____  / / /_/ /  /_/ /_(__  )/ /_ _  /   /  __/ /_/ /_  / / / / /
...  ___  /  \____/_\__, / /____/ \__/ /_/    \___/\__,_/ /_/ /_/ /_/
...  /___/         /____/
... joystream:runtime:base Init
... joystream:runtime:identities Init
... joystream:runtime:identities Initializing key from /root/<5YourStorageAddress.json>
... joystream:runtime:identities Successfully initialized with address <5YourStorageAddress>
... joystream:runtime:balances Init
... joystream:runtime:roles Init
... joystream:runtime:assets Init
... joystream:runtime:system Init
... joystream:runtime:base Waiting for chain to be synced before proceeding.
... joystream:colossus Fetching data objects
... joystream:colossus:api:asset created path handler
... [HPM] Proxy created: function (path, req) {
...   // we get the full path here so it needs to match the path where
...   // it is used by the openapi initializer
...   return path.match('^/asset/v0') && (req.method === 'GET' || req.method === 'HEAD')
... }  -> http://localhost:8080/
... Starting API server...
... API server started. { address: '::', family: 'IPv6', port: 3000 }
... joystream:storage:storage IPFS node is up with identity: <ipfsPeerId>
... joystream:colossus announcing public url
... joystream:runtime:base:tx Submitted: {"nonce":"<nonce>","txhash":"<txhash>","tx":"<tx>"}
```

If everything is working smoothly, you will now start syncing the `content directory`.

### Check that you are syncing
After you've had it running for a bit (>1 min):
```
$ cd ~/joystream/
$ yarn run helios
```
If everything is working, you should rather quickly, see your SP as active, with correct `workerId` and URL.

Then paste the following in your browser:
`https://<your.cool.url>/storage/swagger.json`
Which should return a json.

That means Colossus, ipfs and your hosting are all working!

### Set an Empty Storage URL

An important to remember for the Storage Providers is that stopping and starting their service, or being "active" during the long sync process, will "ruin" the user experience for content consumers and creators, if not done correctly. The former group will not get the assets they want, and the latter may see their uploads fail, if your is randomly selected as the liason, despite being offline.

To avoid doing this, which should get you fired, or at the very least a warning, you have to set an empty `public-url`:

The easiest way of doing so fast:

```
$ cd ~/joystream
$ DEBUG=joystream:* yarn run colossus server --key-file <5YourStorageAddress.json> --public-url  --provider-id <your_storage-id>
```
You can (should) kill the process as soon as you see the line below in the log.
```
joystream:colossus announcing public url
```

Verify you succeeded, by waiting a couple minutes, then:
```
$ cd ~/joystream
$ yarn run helios

# should for <your_storage_id> produce:
<your_storage_id> No url set, skipping
```

### Run storage node as a service

To ensure high uptime, it's best to set the system up as a `service`. Note that this will not work if you set a password for your `<5YourStorageAddress.json> `.

Example file below:

```
$ nano /etc/systemd/system/storage-node.service
# Paste in everything below the stapled line
---
[Unit]
Description=Joystream Storage Node
After=network.target ipfs.service joystream-node.service

[Service]
User=root
WorkingDirectory=/root/joystream/storage-node
LimitNOFILE=10000
Environment=DEBUG=joystream:*,-joystream:util:ranges
ExecStart=/root/.volta/bin/node \
        packages/colossus/bin/cli.js \
        --key-file <5YourStorageAddress.json> \
        --public-url \
#        --public-url https://<your.cool.url>/storage/ \
        --provider-id <your_storage_id>
Restart=on-failure
StartLimitInterval=600

[Install]
WantedBy=multi-user.target
```
**Note**
Before you have completed syncing the content directory, comment out the line with your "actual" `--public-url` as shown in the examples, eg:

```
        --public-url \
#        --public-url https://<your.cool.url>/storage/ \
```


 More [here](#enable-the-public-url-when-synced)

Save and exit. Close `colossus` if it's still running, then:
```
$ systemctl start storage-node
# If everything works, you should get no output.

# Verify with:
$ systemctl status storage-node

# Which, if you have NOT finished syncing, should produce something like:
---
● storage-node.service - Joystream Storage Node
...
... joystream:colossus announcing complete.
... joystream:storage:storage Pinning hash: <ipfsId> content-id: <contentId>
... joystream:sync Sync run completed, set 0 new relationships to ready
... joystream:storage:storage Pinned <ipfsId>
... joystream:sync Creating new storage relationship for <joystreamAddress>
...
---

# If you HAVE finished syncing, and is restarting, it should produce something like:
---
● storage-node.service - Joystream Storage Node
...
... joystream:colossus Fetching data objects
... joystream:sync objects syncing: 191
... joystream:sync objects local: 4837
... joystream:colossus Fetching data objects
... joystream:sync objects syncing: 180
... joystream:sync objects local: 5182
... joystream:colossus Fetching data objects
...
---

# To have colossus start automatically at reboot:
$ systemctl enable storage-node
# If you want to stop the storage node, either to edit the storage-node.service file or some other reason:
$ systemctl stop storage-node
```

### Enable the Public URL When Synced
After your node is fully synced, which you will know by checking the logs of Colossus, and seeing no, or just a few, items still syncing:
```
$ journalctl -f -n 200 -u storage-node.service

# Which should return something like:
... joystream:sync objects syncing: 0
... joystream:sync objects local: 11915
```

Note `objects syncing: 0`, or some very low number, like 1 or 2
Note `objects local: 11915`, where you compare the number to the (extended) output of `helios`:
```
$ cd ~/joystream
$ yarn run helios

# which will (after a minute or so) show:

Found <n> staked providers

...

<id> <last.storageprovider.url/storage/ - OK

Data Directory objects:
12540 created
11915 accepted
11137 unique accepted hashes

...

# You can kill the process after this.
```
If the number of `accepted` in `helios` matches the number of `joystream:sync objects local` in your log (or is off by a few), you are done syncing!


#### Enable the Public URL
There are two ways of doing this. As it takes quite a long time for a restarted storage-node to be fully operational, the first method, which is a little more difficult, is preferable:

1. Avoiding downtime:
```
$ cd ~/joystream/utils/
$ nano src/stringToBytes.ts

# Edit the constant:
const stringToConvert = "example"

# to:
const stringToConvert = "https://<your.cool.url>/storage/"

# save and exit, then:
$ yarn build && node lib/stringToBytes.js

# which should produce a long hex, 0xmyStorageUrlAsHexEncodedBytes
```
Using the "Extrinsics" tab in [Pioneer](https://testnet.joystream.org/#/extrinsics)
- Select `storageWorkingGroup - updateRoleStorage`
- Choose your role key `5YourStorageAddress`
- Set the inputs `<your_storage_id>` and `0xmyStorageUrlAsHexEncodedBytes`
- Submit

Optionally, as it will minimize the damage if your storage node goes down:
```
$ nano /etc/systemd/system/storage-node.service

# Switch which line is commented out, from:
---
        --public-url \
#        --public-url https://<your.cool.url>/storage/ \
---
to
---
#        --public-url \
        --public-url https://<your.cool.url>/storage/ \
---

# Save and close, then:
$ systemctl daemon-reload
```

### Verify everything is working

Wait a minute or so, then confirm `helios` has your real URL, that you are synced, and have the files you think locally:
```
$ cd ~/joystream
$ yarn run helios

# which will (after a minute or so) show:

Found <n> staked providers

...
<your_storage_id> https://<your.cool.url>/storage/ - OK
...

Data Directory objects:
<x> created
<y> accepted
<z> unique accepted hashes

...

    Final Result for provider <your_storage_id>
    fetched: <y>/<y>
    failed: 0
    took: <t>s

# You can kill the process after this.
```
Hopefully, `<t>` isn't too large of a number, and you are not at the bottom. If that is the case, you may need to upgrade your hardware, and that will require you to go through this again :D

#### Further Checks
In your browser, find and click on an uploaded media file [here](https://testnet.joystream.org//#/media/), then open the developer console, and find the URL of the asset. Copy the `<content-id>`, ie. whatever comes after the last `/`.

Then paste the following in your browser:
`https://<your.cool.url>/storage/swagger.json`
Which should return a json.

And:
`https://<your.cool.url>/storage/asset/v0/<content-id>`.
(eg. `5ESiKRXVJGmXvRmgyFRR7QPnbXj29GjJ5DE3ZragDySPMUWJ`)
If the content starts playing, that means you are good!

## Update Your Storage Node

If you have been running a Storage Node on previous network, but not yet on `antioch/sumer`, go [here](#update-older-storage-nodes).

If you have been running a Storage Node on `antioch/sumer`:
- First, update IPFS [here](#update-ipfs)
- Then, continue below

### Update Colossus

An important to remember for the Storage Providers is that stopping and starting their service will "ruin" the user experience for content consumers and creators, if not done correctly. The former group will not get the assets they want, and the latter may see their uploads fail, if your is randomly selected as the liason, despite being offline.

#### Change Your Storage URL
This can be done in one of two ways:

1. Using the "Extrinsics" tab in [Pioneer](https://testnet.joystream.org/#/extrinsics)
- Select `storageWorkingGroup - updateRoleStorage`
- Choose your role key `5YourStorageAddress`
- Set the inputs `<your_storage_id>` and `0x00`
- Submit

2. Restart `colossus` with an empty `--public-url`.

If you are running as a service:
```
$ nano /etc/systemd/system/storage-node.service

# then, delete the URL, leaving you with:
...
  --public-url \
...
# save and exit

$ systemctl daemon-reload
$ systemctl restart storage-node
```

In either case, wait a couple minutes, then verify with:
```
$ cd ~/joystream
$ yarn run helios


# If you passed an empty string (option 1):
<your_storage_id> No url set, skipping

# If you set it `0x00` in pioneer (option 2):
<your_storage_id> - Request path contains unescaped characters
```

If your actual `public-url` still shows up, wait a little longer or try again.

#### Get the updated version

```
# If you are running as service (which you should)
$ systemctl stop storage-node
$ cd ~/joystream/storage-node/packages/colossus
$ yarn unlink
$ cd ~/joystream
$ git pull origin master
$ rm -rf node modules
$ yarn cache clean
$ ./setup.sh
# this requires you to start a new session. if you are using a vps:
$ exit
$ ssh user@ipOrURL
# on your local machine, just close the terminal and open a new one
$ cd ~/joystream
$ yarn build:packages
$ cd ~/joystream/storage-node/packages/colossus
$ yarn run colossus --help
```

#### Start Up
Now, go [here](#run-storage-node-as-a-service).

## Update Older Storage Nodes
If you haven't been running a node in a long time, it's probably best to start fresh. If you don't want to, a quick guide, that may get you into trouble, can be found below:

### Update IPFS and Caddy

It's probably time to update `caddy` (although older versions _should_ continue to work). Go [here](#setup-hosting).

It's definitely time to update `ipfs` (older version will probably work, but you will likely get fired as `helios` won't work correctly).

If you, for some reason is still running them:
```
$ systemctl stop ipfs.service && systemctl stop storage-node.service
```
Delete your IPFS repo, with: `rm -rf ~/.ipfs`. You will save a **lot** of storage space, as the content-directory wasn't migrated.

How to install the latest version of IPFS can be found [here](#install-ipfs).

### Update Colossus

```
# If you are running as service (which you should)
$ systemctl stop storage-node
$ cd ~/joystream/storage-node/packages/colossus
$ yarn unlink
$ cd ~/joystream
$ git pull origin master
$ rm -rf node modules
$ yarn cache clean
$ ./setup.sh
# this requires you to start a new session. if you are using a vps:
$ exit
$ ssh user@ipOrURL
# on your local machine, just close the terminal and open a new one
$ cd ~/joystream
$ yarn build:packages
$ cd ~/joystream/storage-node/packages/colossus
$ yarn run colossus --help
```

#### Optional Cleanup

If you have been running a storage node previously, and used `.bash_profile` to avoid the `yarn run` prefix, you need to:
`$ nano ~/.bash_profile`
Then, comment out, or simple delete, the lines below:
```
# Colossus
alias colossus="/root/storage-node-joystream/packages/colossus/bin/cli.js"  
alias helios="/root/storage-node-joystream/packages/helios/bin/cli.js"
```

### Start Up
Now, go [here](#run-storage-node-as-a-service).

# Troubleshooting
If you had any issues setting it up, you may find your answer here!

## Port not set

If you get an error like this:
```
Error: listen EADDRINUSE: address already in use :::3000
```

It most likely means your port is blocked. This could mean your storage-node is already running (in which case you may want to kill it unless it's configured as a service), or that another program is using the port.

In case of the latter, you can specify a new port (e.g. 3001) with the `--port 3001` flag.
Note that you have to modify the `Caddyfile` as well...

## No tokens in role account
If you try to run the storage-node without tokens to pay the transaction fee, you may at some point have tried so many times your transaction gets "temporarily banned". In this case, you either have to wait for a while, or use the [CLI](/tools/cli/README.md#working-groups:updateRoleAccount) tool to change your "role account".


## Caddy v1 (deprecated)

These instructions below are for Caddy v1. If you don't already have it installed, it will not work. The instructions are only kept in case you happen to have installed it on your computer/VPS already.

```
$ curl https://getcaddy.com | bash -s personal
# Allow caddy access to required ports:
$ setcap 'cap_net_bind_service=+ep' /usr/local/bin/caddy
$ ulimit -n 8192
```

Configure caddy with `nano ~/Caddyfile` and paste in the following:

```
# Storage Node API
https://<your.cool.url> {
    proxy / localhost:3000 {
        transparent
    }
    header / {
        Access-Control-Allow-Origin  *
        Access-Control-Allow-Methods "GET, PUT, HEAD, OPTIONS"
    }
}
```
Now you can check if you configured correctly, with:
```
$ /usr/local/bin/caddy --validate --conf ~/Caddyfile
# Which should return:
Caddyfile is valid

# You can now run caddy with:
$ (screen) /usr/local/bin/caddy --agree --email <your_mail@some.domain> --conf ~/Caddyfile
```
After a short wait, you should see:
```
YYYY/MM/DD HH:NN:SS [INFO] [<your.cool.url>] Server responded with a certificate.
done.

Serving HTTPS on port 443
https://<your.cool.url>


Serving HTTP on port 80
https://<your.cool.url>

```

### Run caddy as a service
To ensure high uptime, it's best to set the system up as a `service`.

Example file below:

```
$ nano /etc/systemd/system/caddy.service
# Paste in everything below the stapled line
---
[Unit]
Description=Reverse proxy for storage node
After=network.target

[Service]
User=root
WorkingDirectory=/root
LimitNOFILE=8192
PIDFile=/var/run/caddy/caddy.pid
ExecStart=/usr/local/bin/caddy -agree -email <your_mail@some.domain> -pidfile /var/run/caddy/caddy.pid -conf /root/Caddyfile
Restart=on-failure
StartLimitInterval=600


[Install]
WantedBy=multi-user.target
```
Save and exit. Close `caddy` if it's still running, then:
```
$ systemctl start caddy
# If everything works, you should get an output. Verify with:
$ systemctl status caddy
# Which should produce something like:
---
● caddy.service - Reverse proxy for storage node
   Loaded: loaded (/etc/systemd/system/caddy.service; disabled)
   Active: active (running) since Day YYYY/MM/DD HH:NN:SS UTC; 6s ago
 Main PID: 9053 (caddy)
   CGroup: /system.slice/caddy.service
           9053 /usr/local/bin/caddy -agree email <your_mail@some.domain> -pidfile /var/run/caddy/caddy.pid -conf /root/Caddyfile

Mon DD HH:NN:SS localhost systemd[1]: Started Reverse proxy for hosted apps.
Mon DD HH:NN:SS localhost caddy[9053]: Activating privacy features... done.
Mon DD HH:NN:SS localhost caddy[9053]: Serving HTTPS on port 443
Mon DD HH:NN:SS localhost caddy[9053]: https://<your.cool.url>
Mon DD HH:NN:SS localhost caddy[9053]: https://<your.cool.url>
Mon DD HH:NN:SS localhost caddy[9053]: Serving HTTP on port 80
Mon DD HH:NN:SS localhost caddy[9053]: https://<your.cool.url>
---
# To have caddy start automatically at reboot:
$ systemctl enable caddy
# If you want to stop caddy, either to edit the file or some other reason:
$ systemctl stop caddy
```
