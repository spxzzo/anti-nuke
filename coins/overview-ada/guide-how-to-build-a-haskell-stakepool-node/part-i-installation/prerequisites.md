# Prerequisites

## :man\_mage: Mandatory Skills for Stake Pool Operators

As a Stake Pool Operator (SPO) for Cardano, you need:

* Operational knowledge of how to set up, run and maintain a Cardano node continuously
* A commitment to maintain your node 24/7/365
* System operation skills including general knowledge of using [Bash scripts](https://linuxconfig.org/bash-scripting-tutorial-for-beginners), [JavaScript Object Notation (JSON) format](https://attacomsian.com/blog/what-is-json?msclkid=0445ae34ce4d11ec84216d09187b5112), [systemd services](https://linuxconfig.org/how-to-create-systemd-service-unit-in-linux) and [cron jobs](https://itsfoss.com/cron-job/)
* Server administration skills (operational and maintenance)
* Fundamental understanding of [networking](https://www.ibm.com/cloud/learn/networking-a-complete-guide)

## :mage: Mandatory Experience for Stake Pool Operators

* Experience of development and operations (DevOps)
* Experience in how to [harden ](https://www.lifewire.com/harden-ubuntu-server-security-4178243)and [secure a server](https://gist.github.com/lokhman/cc716d2e2d373dd696b2d9264c0287a3).
* In the [Cardano Developer Portal](https://developers.cardano.org/docs/get-started/), successfully complete the section [Operate a Stake Pool](https://developers.cardano.org/docs/operate-a-stake-pool/) including the [Stake Pool Course](https://developers.cardano.org/docs/stake-pool-course/)

{% hint style="danger" %}
:octagonal\_sign: **Before continuing this guide, you must satisfy the above requirements**. :construction:
{% endhint %}

## :reminder\_ribbon: Minimum Stake Pool Hardware Requirements

* **Two separate servers**: 1 block producer node, 1 registered relay node
* **One air-gapped offline machine (cold environment)**
* **Operating system**: 64-bit Linux (i.e. Ubuntu 22.04 LTS)
* **Processor:** An Intel or AMD x86 processor with two or more cores, at 2GHz or faster
* **Memory:** 24GB RAM (including swap space)
* **Storage:** 200GB free storage
* **Internet:** Static IP address and a broadband connection supporting speeds at least 10 Mbps
* **Data Plan**: at least 1GB per hour. 720GB per month.
* **Power:** Reliable electrical power
* **ADA balance:** at least 505 ADA for pool deposit and transaction fees

## :man\_lifting\_weights: Recommended Future-proof Stake Pool Hardware Setup <a href="#futureproof" id="futureproof"></a>

* **Four separate servers**: 1 block producer node, 3 relay nodes (2 registered relays and 1 unregistered relay)
* **One air-gapped offline machine (cold environment)**
* **Operating system**: 64-bit Linux (i.e. Ubuntu 22.04 LTS)
* **Processor:** 4 core or higher CPU
* **Memory**: 24GB+ RAM
* **Storage**: 250GB+ free storage
* **Internet**: Static IP addresses and broadband connections supporting speeds of at least 100 Mbps
* **Data Plan**: Unlimited
* **Power:** Reliable electrical power with UPS
* **ADA balance**: more pledge is better, to be determined by **a0**, the pledge influence factor

## :unlock: Recommended Stake Pool Security

If you need ideas on how to harden your stake pool's nodes, refer to [this guide](hardening-an-ubuntu-server.md).

## :tools: Setup Ubuntu

Refer to the respective guide to [Install Ubuntu Server](https://ubuntu.com/tutorials/install-ubuntu-server) or [Install Ubuntu Desktop](https://ubuntu.com/tutorials/install-ubuntu-desktop).

## :bricks: Rebuilding Nodes

If you are rebuilding or reusing an existing `cardano-node` installation, refer to the section [Resetting an Installation](../part-v-tips/resetting-an-installation.md).
