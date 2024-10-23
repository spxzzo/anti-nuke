# Step 4: Installing execution client

## Pick an execution client

Your choice of either [**Geth**](https://geth.ethereum.org)**,** [**Besu**](https://besu.hyperledger.org)**,** [**Nethermind**](https://www.nethermind.io)**, or** [**Erigon**](https://github.com/ledgerwatch/erigon)**.**

{% hint style="info" %}
To strengthen Ethereum's resilience against potential attacks or consensus bugs, it's best practice to run a minority client in order to increase client diversity. Find the latest distribution of execution clients here: [https://clientdiversity.org](https://clientdiversity.org/)

:shield: **Recommendation** :shield:: Nethermind
{% endhint %}

{% hint style="warning" %}
**Reminder**: Ensure you are logged in and execute all steps in this guide as non-root user, `ethereum ,`created during Step 2: Configuring Node.
{% endhint %}

{% tabs %}
{% tab title="Geth" %}
{% hint style="info" %}
**Geth** - Go Ethereum is one of the three original implementations (along with C++ and Python) of the Ethereum protocol. It is written in **Go**, fully open source and licensed under the GNU LGPL v3.
{% endhint %}



Review the latest release notes at [https://github.com/ethereum/go-ethereum/releases](https://github.com/ethereum/go-ethereum/releases)



:dna:**Install from the repository**

```
sudo add-apt-repository -y ppa:ethereum/ethereum
sudo apt-get update -y
sudo apt-get install ethereum -y
```



:gear: **Setup and configure systemd**



Run the following to create a **unit file** to define your `eth1.service` configuration.

Simply copy/paste the following.



```bash
cat > $HOME/eth1.service << EOF
[Unit]
Description=Geth Execution Layer Client service
Wants=network-online.target
After=network-online.target
Documentation=https://www.coincashew.com

[Service]
Type=simple
User=ethereum
Restart=on-failure
RestartSec=3
TimeoutSec=690
ExecStart=/usr/bin/geth \
  --mainnet \
  --metrics \
  --pprof \
  --authrpc.jwtsecret=/secrets/jwtsecret

[Install]
WantedBy=multi-user.target
EOF
```



Move the unit file to `/etc/systemd/system` and give it permissions.

```bash
sudo mv $HOME/eth1.service /etc/systemd/system/eth1.service
```

```bash
sudo chmod 644 /etc/systemd/system/eth1.service
```



Run the following to enable auto-start at boot time.

```
sudo systemctl daemon-reload
sudo systemctl enable eth1
```



:chains:**Start geth**

```
sudo systemctl start eth1
```
{% endtab %}

{% tab title="Besu" %}
{% hint style="info" %}
**Hyperledger Besu** is an open-source Ethereum client designed for demanding enterprise applications requiring secure, high-performance transaction processing in a private network. It's developed under the Apache 2.0 license and written in **Java**.
{% endhint %}



:dna:**Install java dependency**

```
sudo apt update
sudo apt install openjdk-17-jre -y libjemalloc-dev jq
```



:last\_quarter\_moon\_with\_face:**Download and unzip Besu**



Review the latest release at [https://github.com/hyperledger/besu/releases](https://github.com/hyperledger/besu/releases)



Run the following to automatically download the latest linux release, un-tar and cleanup.

```bash
RELEASE_URL="https://api.github.com/repos/hyperledger/besu/releases/latest"
TAG=$(curl -s $RELEASE_URL | jq -r .tag_name)
BINARIES_URL="https://github.com/hyperledger/besu/releases/download/$TAG/besu-$TAG.tar.gz"

echo Downloading URL: $BINARIES_URL

cd $HOME
wget -O besu.tar.gz "$BINARIES_URL"
tar -xzvf besu.tar.gz -C $HOME
rm besu.tar.gz && mv besu-* besu
```



:gear: **Setup and configure systemd**



Run the following to create a **unit file** to define your `eth1.service` configuration.

Simply copy/paste the following.

```bash
cat > $HOME/eth1.service << EOF
[Unit]
Description=Besu Execution Layer Client service
Wants=network-online.target
After=network-online.target
Documentation=https://www.coincashew.com

[Service]
Type=simple
User=ethereum
Restart=on-failure
RestartSec=3
KillSignal=SIGINT
TimeoutStopSec=690
Environment="JAVA_OPTS=-Xmx5g"
ExecStart=/home/ethereum/besu/bin/besu \
  --network=mainnet \
  --metrics-enabled=true \
  --sync-mode=X_CHECKPOINT \
  --data-storage-format=BONSAI \
  --data-path="/home/ethereum/.besu" \
  --Xplugin-rocksdb-high-spec-enabled \
  --engine-jwt-secret=/secrets/jwtsecret

[Install]
WantedBy=multi-user.target
EOF
```



{% hint style="info" %}
Good to know: --Xplugin-rocksdb-high-spec-enabled allows Besu increased database performance. Recommended for machines with 16GB of RAM or more.
{% endhint %}



Move the unit file to `/etc/systemd/system` and give it permissions.

```bash
sudo mv $HOME/eth1.service /etc/systemd/system/eth1.service
```

```bash
sudo chmod 644 /etc/systemd/system/eth1.service
```



Run the following to enable auto-start at boot time.

```
sudo systemctl daemon-reload
sudo systemctl enable eth1
```



:chains: **Start besu**

```
sudo systemctl start eth1
```
{% endtab %}

{% tab title="Nethermind" %}
{% hint style="info" %}
**Nethermind** is a flagship Ethereum client all about performance and flexibility. Built on **.NET** core, a widespread, enterprise-friendly platform, Nethermind makes integration with existing infrastructures simple, without losing sight of stability, reliability, data integrity, and security.
{% endhint %}



:gear: **Install dependencies**

```
sudo apt-get update
sudo apt-get install curl libsnappy-dev libc6-dev jq libc6 unzip -y
```



:last\_quarter\_moon\_with\_face:**Download and unzip Nethermind**



Review the latest release at [https://github.com/NethermindEth/nethermind/releases](https://github.com/NethermindEth/nethermind/releases)

Run the following to automatically download the latest linux release, un-zip and cleanup.

```bash
cd $HOME
curl -s https://api.github.com/repos/NethermindEth/nethermind/releases/latest | jq -r ".assets[] | select(.name) | .browser_download_url" | grep linux-x64 | xargs wget -q --show-progress
unzip -o nethermind*.zip -d $HOME/nethermind
rm nethermind*linux*.zip
```



:gear: **Setup and configure systemd**



Run the following to create a **unit file** to define your `eth1.service` configuration.

Simply copy/paste the following.

```bash
cat > $HOME/eth1.service << EOF
[Unit]
Description=Nethermind Execution Layer Client service
Wants=network-online.target
After=network-online.target
Documentation=https://www.coincashew.com

[Service]
Type=simple
User=ethereum
Restart=on-failure
RestartSec=3
KillSignal=SIGINT
TimeoutStopSec=690
WorkingDirectory=/home/ethereum/nethermind
ExecStart=/home/ethereum/nethermind/Nethermind.Runner \
  --baseDbPath /home/ethereum/.nethermind \
  --Metrics.Enabled true \
  --Metrics.ExposePort 6060 \
  --Metrics.IntervalSeconds 10000 \
  --Sync.SnapSync true \
  --Sync.AncientBodiesBarrier 11052984 \
  --Sync.AncientReceiptsBarrier 11052984 \
  --JsonRpc.JwtSecretFile /secrets/jwtsecret

[Install]
WantedBy=multi-user.target
EOF
```



Move the unit file to `/etc/systemd/system` and give it permissions.

```bash
sudo mv $HOME/eth1.service /etc/systemd/system/eth1.service
```

```bash
sudo chmod 644 /etc/systemd/system/eth1.service
```



Run the following to enable auto-start at boot time.

```
sudo systemctl daemon-reload
sudo systemctl enable eth1
```



:chains: **Start Nethermind**

```
sudo systemctl start eth1
```
{% endtab %}

{% tab title="Erigon" %}
{% hint style="info" %}
**Erigon** - Successor to OpenEthereum, Erigon is an implementation of Ethereum (aka "Ethereum client"), on the efficiency frontier, written in Go.
{% endhint %}

{% hint style="warning" %}
Erigon requires at least 16GB RAM. 32GB is highly recommended.
{% endhint %}

:gear: **Install Go dependencies**

Find the [latest release of Go from here ](https://go.dev/doc/install)and update the **download URL** for the **tar.gz file** below.

```
wget -O go.tar.gz https://go.dev/dl/go1.19.linux-amd64.tar.gz
```

```bash
sudo rm -rf /usr/local/go && sudo tar -C /usr/local -xzf go.tar.gz
```

```bash
echo export PATH=$PATH:/usr/local/go/bin>> $HOME/.bashrc
source $HOME/.bashrc
```

Verify Go is properly installed and cleanup files.

```bash
go version
rm go.tar.gz
```

:robot: **Build and install Erigon**

Install build dependencies.

```bash
sudo apt-get update
sudo apt install build-essential git
```

Review the latest release at [https://github.com/ledgerwatch/erigon/releases](https://github.com/ledgerwatch/erigon/releases)

```bash
cd $HOME
git clone --branch stable --recurse-submodules https://github.com/ledgerwatch/erigon.git
cd erigon
make erigon
```

​Verify the build completed successfully by displaying Erigon's version.

```bash
./build/bin/erigon --version
```

​ Example of version:

```
erigon version 2.47.0-stable-5536d645
```

Make data directory and update directory ownership.

```bash
sudo mkdir -p /var/lib/erigon
sudo chown ethereum:ethereum /var/lib/erigon
```

:gear: **Setup and configure systemd**



Run the following to create a **unit file** to define your `eth1.service` configuration.

Simply copy/paste the following.

```bash
cat > $HOME/eth1.service << EOF
[Unit]
Description=Erigon Execution Layer Client service
Wants=network-online.target
After=network-online.target
Documentation=https://www.coincashew.com

[Service]
Type=simple
User=ethereum
Restart=on-failure
RestartSec=3
KillSignal=SIGINT
TimeoutStopSec=690
ExecStart=/home/ethereum/erigon/build/bin/erigon \
 --datadir /var/lib/erigon \
 --chain mainnet \
 --metrics \
 --pprof \
 --prune htc \
 --prune.r.before=11052984 \
 --authrpc.jwtsecret=/secrets/jwtsecret

[Install]
WantedBy=multi-user.target
EOF
```



{% hint style="info" %}
By default with Erigon, `--prune` deletes data older than 90K blocks from the tip of the chain. For example, if tip block is no. 12'000'000, then only the data between 11'910'000-12'000'000 will be kept).
{% endhint %}



Move the unit files to `/etc/systemd/system` and give it permissions.

```bash
sudo mv $HOME/eth1.service /etc/systemd/system/eth1.service
```

```bash
sudo chmod 644 /etc/systemd/system/eth1.service
```



Run the following to enable auto-start at boot time.

```
sudo systemctl daemon-reload
sudo systemctl enable eth1
```



:chains:**Start Erigon**

```
sudo systemctl start eth1
```
{% endtab %}
{% endtabs %}

### :tools: Helpful execution client commands

{% tabs %}
{% tab title="View logs" %}
```
journalctl -fu eth1
```
{% endtab %}

{% tab title="Stop" %}
```
sudo systemctl stop eth1
```
{% endtab %}

{% tab title="Start" %}
```
sudo systemctl start eth1
```
{% endtab %}

{% tab title="View Status" %}
```
sudo systemctl status eth1
```
{% endtab %}
{% endtabs %}

### :ladder: Next Steps

* Now that your execution client is configured and started, proceed to the [next section to setup your consensus client.](broken-reference)

{% hint style="warning" %}
If you're checking the logs and see any warnings or errors, please be patient as these will normally resolve once both your execution and consensus clients are fully synched to the Ethereum network.
{% endhint %}
