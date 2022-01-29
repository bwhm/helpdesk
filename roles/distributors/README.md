<p align="center"><img src="img/distributor.png"></p>

<div align="center">
  <h4>This is a guide to setting up your <a href="https://github.com/Joystream/joystream/tree/master/distributor-node">distributor node</a>, and getting started as a Distributor Provider on the latest
    <a href="https://testnet.joystream.org/">testnet</a>.</h4><br>
</div>



Table of Contents
==
<!-- TOC START min:1 max:3 link:true asterisk:false update:true -->
- [Overview](#overview)
  - [Resources](#resources)
- [Instructions](#instructions)
  - [Initial setup](#initial-setup)
  - [Setup Hosting](#setup-hosting)
    - [Caddy](#caddy)
    - [Run caddy as a service](#run-caddy-as-a-service)
  - [Setup Query Node](#setup-query-node)
  - [Install and Setup the Distributor Node](#install-and-setup-the-distributor-node)
    - [Applying for a Distributor opening](#applying-for-a-distributor-opening)
    - [Setup and Configure the Distributor Node](#setup-and-configure-the-distributor-node)
    - [Config File](#config-file)
    - [Accept Invitation](#accept-invitation)
    - [Set Metadata](#set-metadata)
  - [Deploy the Distributor Node](#deploy-the-distributor-node)
    - [Verify everything is working](#verify-everything-is-working)
- [Troubleshooting](#troubleshooting)
<!-- TOC END -->


# Overview

This page contains all information required to set up your distributor node and become a `Distributor Provider` on the current Joystream testnet.

The guide for the `Distributor Provider Lead` can be found [here](/roles/distributors-lead).

## Resources
To understand how the distributor node works in detail, checkout the [node](https://github.com/Joystream/joystream/blob/master/distributor-node/docs/node/index.md) documentation.

You may also find it helpful to look at the [Distributor Lead guide](/roles/distributors-lead), and the [gitbook](https://joystream.gitbook.io/testnet-workspace/).

# Instructions

The instructions below will assume you are running as `root`. This makes the instructions somewhat easier, but less safe and robust.

Note that this has been tested on a fresh images of `Ubuntu 20.04 LTS`.

The system has shown to be less resource intensive, so you may get away with a VPS with specs equivalent to [Linode 4GB](https://www.linode.com/pricing/) or better (not an affiliate link).

Please note that unless there are any openings for new distributor providers (which you can check in [Pioneer](https://testnet.joystream.org/) under `Working Groups` -> `Opportunities`), you will not be able to join. Applying to the opening is easiest in Pioneer, but once hired, you no longer need it. Actions you may want to perform after getting hired are easiest to carry out with the [CLI](/tools/cli/README.md#working-groups). With this, you can configure things like:
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
```

## Setup Hosting
In order to allow for users to upload and download, you have to setup hosting, with an actual domain as both Chrome and Firefox requires `https://`. If you have a "spare" domain or subdomain you don't mind using for this purpose, go to your domain registrar and point your domain to the IP you want. If you don't, you will need to purchase one.

### Caddy
To configure SSL-certificates the easiest option is to use [caddy](https://caddyserver.com/), but feel free to take a different approach. Note that if you are using caddy for commercial use, you need to acquire a license. Please check their terms and make sure you comply with what is considered personal use.

For the best setup, you should use the "official" [documentation](https://caddyserver.com/docs/).

The instructions below are for Caddy v2.4.1:
```
$ wget https://github.com/caddyserver/caddy/releases/download/v2.4.6/caddy_2.4.6_linux_amd64.tar.gz
$ tar -vxf caddy_2.4.6_linux_amd64.tar.gz
$ mv caddy /usr/bin/
# Test that it's working:
$ caddy version
```

Configure the `Caddyfile`:
```
$ nano ~/Caddyfile
# Modify, and paste in everything below the stapled line
---
# Joystream-node
wss://<your.cool.url>/rpc {
	reverse_proxy localhost:9944
}


# Distributor Node
https://<your.cool.url>/distributor/* {
        log {
                output stdout
        }
        route /distributor/* {
                uri strip_prefix /distributor
                reverse_proxy localhost:3334
        }
        header /distributor {
                Access-Control-Allow-Methods "GET, PUT, HEAD, OPTIONS, POST"
                Access-Control-Allow-Headers "GET, PUT, HEAD, OPTIONS, POST"
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

# Modify, and paste in everything below the stapled line
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
# If you want to edit your Caddfile, edit it, then run:
$ caddy reload
```

## Setup Query Node
[Go here for the installation guide](/tools/query-node/README.md).

## Install and Setup the Distributor Node

First, you need to clone the Joystream/joystream repo, which contains the distributor software.

```
$ git clone https://github.com/Joystream/joystream.git
$ cd joystream
$ ./setup.sh
# this requires you to start a new session. if you are using a vps:
$ exit
$ ssh user@ipOrURL
$ cd joystream
$ ./build-packages
$ yarn joystream-distributor --help
```

### Applying for a Distributor opening

Click [here](https://testnet.joystream.org) to open the `Pioneer app` in your browser. Then follow instructions [here](https://github.com/Joystream/helpdesk#get-started) to generate a set of `Keys`, get tokens, and sign up for a `Membership`. This `key` will be referred to as the `member` key.

Make sure to save the `5YourJoyMemberAddress.json` file. This key will require tokens to be used as stake for the `Distributor Provider` application (`application stake`) and further stake may be required if you are selected for the role (`role stake`).

To check for current openings, visit [this page](https://testnet.joystream.org/#/working-groups/opportunities) on Pioneer and look for any `Distributor Provider` applications which are open for applications. If there is an opening available, fill in the details requested in the form required and stake the tokens needed to apply (when prompted you can sign a transaction for this purpose).

During this process you will be provided with a role key, which will be made available to download in the format `5YourDistributorRoleKey.json`. If you set a password for this key, remember it :)

The next steps (below) will only apply if you are a successful applicant.


### Setup and Configure the Distributor Node

On the machine/VPS you want to run your distributor node:

```
$ mkdir ~/keys/
```

Assuming you are running the distributor node on a VPS via ssh, on your local machine:
```
# Go the directory where you saved your <5YourDistributorRoleKey.json>, then rename it to

distributor-role-key.json

$ scp distributor-role-key.json <user>@<your.vps.ip.address>:/root/keys/
```

**Make sure your [Joystream full node](https://github.com/Joystream/joystream/releases) and [Query Node](#setup-query-node) is fully synced before you move to the next step(s)!**

### Config File
The default `config.yml` file can be found below. Note that you only need to modify a few lines.
```
nano ~/joystream/distributor-node/config.yml
---

id: test-node
endpoints:
  queryNode: http://localhost:8081/graphql
  joystreamNodeWs: ws://localhost:9944
directories:
  assets: ./local/data
  cacheState: ./local/cache
logs:
  file:
    level: debug
    path: ./local/logs
    maxFiles: 30 # 30 days or 30 * 50 MB
    maxSize: 50485760 # 50 MB
  console:
    level: verbose
  # elastic:
  #   level: info
  #   endpoint: http://localhost:9200
limits:
  storage: 100G
  maxConcurrentStorageNodeDownloads: 100
  maxConcurrentOutboundConnections: 300
  outboundRequestsTimeoutMs: 5000
  pendingDownloadTimeoutSec: 3600
  maxCachedItemSize: 1G
intervals:
  saveCacheState: 60
  checkStorageNodeResponseTimes: 60
  cacheCleanup: 60
publicApi:
  port: 3334
operatorApi:
  port: 3335
  hmacSecret: this-is-not-so-secret
keys:
  - suri: //Alice
  # - mnemonic: "escape naive annual throw tragic achieve grunt verify cram note harvest problem"
  #   type: ed25519
  # - keyfile: "/path/to/distributor-role-key.json"
workerId: 0
```
The following lines must be changed:
```
# Comment out:
  - suri: //Alice
# uncomment and edit
  - keyfile: "/root/keys/keyfile.json"

# replace 0 with your <workerId>
workerId: 0

# replace with a real secret
hmacSecret: this-is-not-so-secret
```

- `endpoints:` If you are not running your own node and/or query node:
  - change both to working endpoints
- `limits:` These numbers should depend on decisions made by the Lead
- `hmacSecret: this-is-not-so-secret`
- `directories:` You may want to change these. Especially if you are renting extra storage volumes
  - `/path/to/storage/volume`

### Accept Invitation
Once hired, the Distributor Lead will invite you a to "bucket". Before this is done, you will not be able to participate. Assuming:
- your Worker ID is `<workerId>`
- the Lead has invited to bucket family `<bucketFamilyId>` with index `<bucketId>` -> `<bucketFamilyId>:<bucketId>`

```
$ cd ~/joystream/distributor-node
$ yarn joystream-distributor operator:accept-invitation -B <bucketFamilyId>:<bucketId> -w <workerId>
```

### Set Metadata
When you have accepted the invitation, you have to set metadata for your node. If your VPS is in Frankfurt, Germany:

```
$ nano ~/joystream/distributor-node/metadata.json
# Modify, and paste in everything below the stapled line
---
{
  "endpoint": "https://<your.cool.url>/distributor/",
  "location": {
    "countryCode": "DE",
    "city": "Frankfurt",
    "coordinates": {
      "latitude": 52,
      "longitude": 15
    }
  },
  "extra": "Hello world!"
}
```
Where:
- The location should really be correct (you can google your way to latitude/longitude)
- extra is not that critical. It could perhaps be nice to add some info on your max capacity?

Then set it on-chain with:

```
$ cd ~/joystream/distributor-node
$ yarn joystream-distributor operator:set-metadata -B <bucketFamilyId>:<bucketId> -w <workerId> -i /path/to/metadata.json
```

## Deploy the Distributor Node
First, create a `systemd` file. Example file below:

```
$ nano /etc/systemd/system/distributor-node.service

# Modify, and paste in everything below the stapled line
---
[Unit]
Description=Joystream Distributor Node
After=network.target joystream-node.service

[Service]
User=root
WorkingDirectory=/root/joystream/
LimitNOFILE=10000
ExecStart=/root/.volta/bin/yarn joystream-distributor start \
        -c /root/joystream/distributor-node/config.yml
Restart=on-failure
StartLimitInterval=600

[Install]
WantedBy=multi-user.target
```

To start the node:

```
$ systemctl start distributor-node
# If everything works, you should get an output. Verify with:
$ journalctl -f -n 200 -u distributor-node

# If it looks ok, it probably is :)
---

# To have the distributor-node start automatically at reboot:
$ systemctl enable distributor-node
# If you want to stop the distributor node, either to edit the distributor-node.service file or some other reason:
$ systemctl stop distributor-node
```

### Verify everything is working

In your browser, try:
`https://<your.cool.url>/distributor/api/v1/status`.

# Troubleshooting
If you had any issues setting it up, you may find your answer here!
