---
description: >-
  Problem: My validator is running behind a dynamic IP address and this causes
  connectivity issues with other validators when my dynamic IP address changes.
---

# Setting up dynamic DNS (DDNS)

## :fast\_forward: Quick steps guide

{% hint style="info" %}
The following steps align with our [mainnet guide](./). You may need to adjust file names and directory locations where appropriate. The core concepts remain the same.
{% endhint %}

### :dagger: Why do I need a dynamic DNS service?

* Most internet connections are through a dynamic IP address and can change weekly or even daily.
* Frequent IP changes can make it difficult to host services such as a validator.
* As a workaround, you can use a DDNS (Dynamic DNS) service.
* Using a subdomain (i.e. mySubDomain.duckdns.org), you relate this to your latest dynamic IP address.
* Periodically, say every 5 minutes, your computer updates a subdomain with your latest dynamic IP address.
* Other validators or users would find you via the subdomain, instead of IP address.

{% hint style="info" %}
There are many [alternative DDNS services](https://hackerspad.net/software/duck-dns/#alternatives) but seldom do they accept crypto donations like [Duck DNS](https://www.duckdns.org).
{% endhint %}

### :robot: Minimum System Requirements

* Linux cron

### :construction: How to Configure the DDNS

For the purpose of this tutorial, we will be using [DuckDNS.org](https://www.duckdns.org/install.jsp)

1\. Sign in and create an account with your preferred social media login.

2\. Follow the instructions on[ how to setup duckdns for linux cron.](https://www.duckdns.org/install.jsp)

3\. Configure the **beacon-chain** to use your new DDNS subdomain.

{% tabs %}
{% tab title="Prysm" %}
```bash
# Edit your beacon-chain unit file
sudo nano /etc/systemd/system/beacon-chain.service

# Append the following flag to ExecStart
--p2p-host-dns <SUBDOMAIN>

# Example of what ExecStart could look like.
# prysm.sh beacon-chain --mainnet --p2p-host-dns mySubDomain.duckdns.org

# Reload the new unit file
sudo systemctl daemon-reload

# Restart your beacon-chain
sudo systemctl restart beacon-chain
```
{% endtab %}
{% endtabs %}

{% hint style="success" %}
Nice work. You're running a DDNS now.&#x20;
{% endhint %}

{% hint style="info" %}
Be sure to familiarize yourself with the [official docs and faqs.](https://www.duckdns.org/faqs.jsp)
{% endhint %}

## :robot: Start staking by building a validator <a href="#start-staking-by-building-a-validator" id="start-staking-by-building-a-validator"></a>

### Visit here for our [Mainnet guide](https://www.coincashew.com/coins/overview-eth/guide-or-how-to-setup-a-validator-on-eth2-mainnet)

{% hint style="success" %}
Congrats on completing the guide. ✨

Did you find our guide useful? Send us a signal with a tip and we'll keep updating it.

It really energizes us to keep creating the best crypto guides.

Use [cointr.ee to find our donation](https://cointr.ee/coincashew) addresses. 🙏

Any feedback and all pull requests much appreciated. 🌛
{% endhint %}

## :jigsaw: Reference Material

{% embed url="https://www.duckdns.org/install.jsp" %}
