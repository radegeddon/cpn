# Chainpoint Node

[![JavaScript Style Guide](https://cdn.rawgit.com/feross/standard/master/badge.svg)](https://github.com/feross/standard)
[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)

## About

Chainpoint Nodes allows anyone to run a server that accepts hashes, anchors them to public
blockchains, create and verify proofs, and participate in the Tierion Network.

Nodes communicate with the Tierion Core, spending TNT to anchor hashes, and gain eligibility to earn TNT by providing services to the Tierion Network.

To be eligible to earn TNT a Node must:

* register a unique Ethereum address
* maintain a minimum balance of 5000 TNT for that address
* provide public network services
* pass all audits and health checks from Tierion Core
* have enough credits to send hashes from a Node to Core

Chainpoint Nodes that don't meet these requirements won't be eligible to earn TNT through periodic rewards.

Chainpoint Nodes aggregate incoming hashes into a Merkle tree every second. The Merkle root is submitted to a Tierion Core for anchoring
to public blockchains.

Nodes maintain a mirror of the Calendar. This allows any Node to verify any proof.

Nodes expose a public HTTP API, documented with Swagger, that you can explore:
[https://app.swaggerhub.com/apis/chainpoint/node/1.0.0](https://app.swaggerhub.com/apis/chainpoint/node/1.0.0)

### Important Notice

This document contains instructions for installing and running
a Chainpoint Node. If you are looking for the full source code
for this application see [github.com/chainpoint/chainpoint-node-src](https://github.com/chainpoint/chainpoint-node-src)

## About the Technology

### How Does It Work?

Chainpoint Node is run as a `docker-compose` application. `docker-compose` is a tool for running multiple Docker containers as
an orchestrated application suite.

`docker-compose` allows distribution of binary images that can run
anywhere that Docker can run. This ensures that everyone, regardless of
platform, is running the same code.

Docker makes it easy to distribute and upgrade the Chainpoint Node
software.

### Software Components

When started, `docker-compose` will install and run three system components in the Docker virtual machine.

* PostgreSQL Database
* Redis
* Chainpoint Node (Node.js)

They are started as a group and should not interfere with any other
software systems running on your server.

Each Node instance you want to run will need:

* A dedicated Ethereum address
* Public IP/hostname
* Minimum TNT balance
* Access to credits, purchased with TNT

### System Requirements

The software should be able to be run on any system that supports
the Docker and Docker Compose container management tools and meets the minimal hardware requirements.

#### Hardware

The minimum hardware requirements for running a Node are
relatively low. The following would be suitable minimums
for a Node expecting relatively light traffic:

- `>= 1GB RAM`
- `1 CPU Core`
- `20GB Hard Disk or SSD`
- `Public IPv4 address`

Running a Node on a server with less than 1GB of RAM has been [known to cause issues](https://github.com/chainpoint/chainpoint-node-src).

If you are expecting larger volumes of hashes to be sent to your server its recommended that you scale-up the system resources by adding more RAM and CPU cores. Alternatively, you can scale-out horizontally by running more Nodes. The disk storage needs of a Node are relatively small.

It is not currently supported to run multiple Nodes on a single
physical host.

#### Operating System

The software has been tested on the following operating systems:

* `Ubuntu 16.04 LTS`
* `macOS Version 10.12.6`

It will likely run on other operating systems that support Docker
and Docker Compose, but support is not currently provided for those.

#### Docker & Docker Compose

Nodes have been developed and tested to run on the following
software versions.

* `Docker version 17.06.2-ce, build cec0b72`
* `docker-compose version 1.14.0, build c7bdf9e`

## Installation

This software is designed to be simple to install and run
on supported systems. Please follow the instructions below
to get started.

For illustrative purposes, we'll provide instructions for running a Node on Digital Ocean, a cloud VPS provider that gives you root access to a host at minimal monthly cost.

### Prerequisites

Before you start, you will need:

* An Ethereum address that you have the private keys for. Exchange provided accounts are generally not supported. You should be able to unlock your account to send Ether or TNT using MyEtherWallet for example.

* You must have the mimimum balance of 5000 TNT to run a Node, and those TNT must be assigned to the Ethereum address you'll
use to identify your Node. You can check your TNT balance
(in Grains, divide by `100000000` (10^8) for TNT balance) using the Etherscan.io
[contract reading tool for our ERC20 smart contract](https://etherscan.io/address/0x08f5a9235b08173b7569f83645d2c7fb55e8ccd8#readContract) (input your address in the `balanceOf` field and click `Query`).

If you run into any issues operating your Node, the first best place to look for help is the [Frequently Asked Questions](https://github.com/chainpoint/chainpoint-node/wiki/Frequently-Asked-Questions) page on our Wiki.

### Start a Server

Your first step is to start a server and gain SSH access
to your account on that server. This is beyond the scope of this document. You will need:

* `root` access, or a user with `sudo` priveleges
* Ubuntu 16.04 OS

Log in to your server and continue to the next step.

### Install Docker and Docker Compose

There are [good instructions available](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-16-04) for installing Docker on an Ubuntu server.

There are also [official docs for installing Docker](https://docs.docker.com/engine/installation/) on other systems.

For some systems you will need to separately install `docker-compose`.

To make this process easier we have created a small script that will install Docker, `docker-compose`, and download this
`chainpoint-node` repository to your system with a single command:

```
curl -sSL https://cdn.rawgit.com/chainpoint/chainpoint-node/58a84a8822b8d79b1041c4fcf1d6e5d8e2fc431a/scripts/docker-install-ubuntu.sh | bash
```

Since this command runs a shell script as a priviledged user on your system we recommend you [examine it carefully](https://github.com/chainpoint/chainpoint-node/blob/master/scripts/docker-install-ubuntu.sh) before you run it.

Simply copy/paste that script into your terminal and it will:

* install Docker
* install Docker Compose
* grant the ability for your local user to run Docker commands without using `sudo`
* download this repository to your home folder.

**Important**: You should close your terminal and login again now to make sure that the changes in the script are fully applied.

### Configure Your Node

Configuration is as simple as editing a single configuration file and providing two variables. We provide a sample configuration file in this repository called `.env.sample`.

The installation script, if you use it, will copy that file
to `~/chainpoint-node/.env` for you, ready to edit.

If you installed everything manually you will want to:

```
cd ~/chainpoint-node
cp .env.sample .env

# use your favorite editor:
vi .env
```

There are only two values that you may need to edit (comments removed for clarity):

```
NODE_TNT_ADDRESS=
CHAINPOINT_NODE_PUBLIC_URI=
```

`NODE_TNT_ADDRESS` : should be set to your Ethereum address that contains TNT balance. It will start with `0x` and have an additional 40 hex characters (`0-9, a-f, A-F`). This is the unique identifier for your Node.

`CHAINPOINT_NODE_PUBLIC_URI` : should be a URI where your Node can be publicly discovered and utilized by others. This might look like `http://10.1.1.20`. Your Node will run on port `80` over `http`. If provided, this address will be periodically audited by Tierion Core to ensure compliance with the rules for a healthy Node. If you leave this config value blank, it will be assumed that your Node is not publicly available, and you will not be eligible to earn TNT rewards.

### Node Firewall

You will need to expose port 80 (HTTP) to the world if you want your Node to be available for
clients to connect to. This is true whether you run behind a non-public host behind a firewall
or publicly. By default, most cloud compute server providers will block all access to your
host's ports by default (usually with the exception of port 22 (SSH)). If your provider
does not implement a block by default policy you are encouraged to choose another provider
or to configure a firewall on your Node server to block access to all ports with the
exception of port 80 HTTP and port 22 SSH.

### Run Your Node

Now its time to start your own Node!

After finishing the configuration in the `.env` file and saving it make sure you are in the `~/chainpoint-node` directory and run `make`. This will show you some Makefile commands that are available to you:

* `make up` : start all services
* `make down` : stop all services
* `make upgrade` : upgrade to newest release
* `make logs` : show, and tail, the `docker-compose` logfiles
* `make ps` : show the status of the running processes

The simplest step at this point is to run `make up`. This
will automatically pull down the appropriate Docker images
(which may take a few minutes the first time, or on slower network systems) and start them. `docker-compose` will also
try to ensure that these services keep running, even if they were to crash (which is unlikely).

### Monitor Your Node

You can run `make ps` to see the services that are running.

You can run `make logs` to tail the logfiles for all `docker-compose` managed services.

When you start your Node you'll see in the logs that your Node will attempt to register itself with one of our Tierion Core clusters and will be provided with a secret key.

The Node will then go through a process of downloading, and cryptographically verifying the entire Calendar. Every block will have its signature checked and will be stored locally. This process may take some time on first run as our Calendar grows. After initial sync, all incremental changes will also be pulled down to every Node, verified and stored.

If there are any problems you see in the logs, or if something is not working as expected, please [file a bug](https://github.com/chainpoint/chainpoint-node/issues) and provide as much information about the issue as possible.

### Transferring TNT for Credits

Update 9/19/2017

Nodes will receive enough credits to submit the maximum number of hashes to Core per day. This credit model will remain active until further notice. At this time, there is no need to convert TNT to credits.

### Sending Hashes to Your Node with the CLI

Now you should be fully up and running! You might want to try out your new Node now with the [Chainpoint CLI](https://github.com/chainpoint/chainpoint-cli).

Normally the CLI will auto-discover a Node to send hashes to. Once you have it installed though, you can configure it to always use your Node if you prefer.

You can either modify the Node address in the `~/.chainpoint/cli.config` to set it permanently, or you can override the Node address every time you use it like this:

```
chp submit --server http://127.0.0.1 <hash>
```

### Stopping Your Node

You can stop your Node at any time with `make down`. You can
verify that everything is stopped with `make ps`.

### Node Authentication Key Backup/Restore

You should understand how Node registration and authentication works. You are strongly encouraged to backup your authentication key(s). The following info may be useful if you need to backup/restore a Node
and run it on another host.

Once your Node starts and registers itself a secret key will be provided for your system and stored in the Node's local database. The Node will use this key to authenticate itself to Tierion Core when
submitting hashes and performing other actions. The database can hold multiple authentication keys
and will choose the right one based on the Ethereum address you've configured in the `NODE_TNT_ADDRESS` environment variable in the `.env` file.

If this secret key is lost, you will likely need to switch to another Ethereum address, and any credits on Tierion Core will be inaccessible. When you first start your Node this secret key is displayed in the logs. You will want to store it somewhere in case of accidental deletion.

Its easy to export your keys at any time by issuing the command `make auth-keys`. This will
select the keys from the database and print them out to the console. Just copy and paste them
somewhere safe.

You can also import an Ethereum address and secret key into your Node's database when restoring
from backup using the following procedure.

* Ensure that the `NODE_TNT_ADDRESS` environment variable in your `.env` file is set to the Ethereum address you want to set an auth key for.
* Run `make auth-key-update KEY=mysecrethexkey` replacing `mysecrethexkey` with the specific hex auth key generated for the `NODE_TNT_ADDRESS` you set.

You can verify that your keys were imported successfully by running `make auth-keys` again.

When you run `make auth-key-update` your Node will be automatically restarted.

On successful restart you should see log messages in `make logs` indicating use of your new auth key.

## License

[Apache License, Version 2.0](https://opensource.org/licenses/Apache-2.0)

```
Copyright (C) 2017 Tierion

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```

## Thank You!

Thank you for being an active participant in the Tierion Network and for your interest in running a Chainpoint Node. We couldn't do it without you!

