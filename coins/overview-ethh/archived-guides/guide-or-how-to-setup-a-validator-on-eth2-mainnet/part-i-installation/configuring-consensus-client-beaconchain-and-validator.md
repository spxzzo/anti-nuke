# Step 5: Installing consensus client

## Pick a consensus client

{% hint style="info" %}
To strengthen Ethereum's resilience against potential attacks or consensus bugs, it's best practice to run a minority client in order to increase client diversity. Find the latest distribution of consensus clients here: [https://clientdiversity.org](https://clientdiversity.org/)

:shield: **Recommendation** :shield:: Lodestar
{% endhint %}

{% hint style="warning" %}
**Reminder**: Ensure you are logged in and execute all steps in this guide as non-root user, `ethereum ,`created during Step 2: Configuring Node.
{% endhint %}

Your choice of [Lighthouse](https://github.com/sigp/lighthouse), [Nimbus](https://github.com/status-im/nimbus-eth2), [Teku](https://consensys.net/knowledge-base/ethereum-2/teku/), [Prysm](https://github.com/prysmaticlabs/prysm) or [Lodestar](https://lodestar.chainsafe.io).

{% tabs %}
{% tab title="Lighthouse" %}
{% hint style="info" %}
[Lighthouse](https://github.com/sigp/lighthouse) is an Eth client with a heavy focus on speed and security. The team behind it, [Sigma Prime](https://sigmaprime.io), is an information security and software engineering firm who have funded Lighthouse along with the Ethereum Foundation, Consensys, and private individuals. Lighthouse is built in Rust and offered under an Apache 2.0 License.
{% endhint %}



:gear: **4.1. Install rust dependency**

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```



Enter '1' to proceed with the default install.



Update your environment variables.

```bash
echo export PATH="$HOME/.cargo/bin:$PATH" >> ~/.bashrc
source ~/.bashrc
```



Install rust dependencies.

```
sudo apt-get update
sudo apt install -y git gcc g++ make cmake pkg-config libssl-dev libclang-dev clang protobuf-compiler
```



:bulb: **4.2. Build Lighthouse from source**

```bash
mkdir ~/git
cd ~/git
git clone -b stable https://github.com/sigp/lighthouse.git
cd lighthouse
make
```



{% hint style="info" %}
Improve some Lighthouse benchmarks by around 20% at the expense of increased compile time? Use `maxperf` profile.\
\
To compile with maxperf, replace the above `make` command with

&#x20; `PROFILE=maxperf make`
{% endhint %}



{% hint style="info" %}
In case of compilation errors, run the following sequence.

```
rustup update
cargo clean
make
```
{% endhint %}



{% hint style="info" %}
This build process may take a few minutes.
{% endhint %}



Verify lighthouse was installed properly by checking the version number.

```
lighthouse --version
```



:tophat: **4.3. Import validator key**

{% hint style="info" %}
When you import your keys into Lighthouse, your validator signing key(s) are stored in the `$HOME/.lighthouse/mainnet/validators` folder.
{% endhint %}



Run the following command to import your validator keys from the eth2deposit-cli tool directory.



Enter your **keystore password** to import accounts.

```bash
lighthouse account validator import --network mainnet --reuse-password --directory=$HOME/staking-deposit-cli/validator_keys
```



Verify the accounts were imported successfully.

```bash
lighthouse account_manager validator list --network mainnet
```



{% hint style="danger" %}
WARNING: Do not import your validator keys into multiple validator clients and run them at the same time, or you might get slashed. If moving validators to a new setup or different validator client, ensure deletion of the previous validator keys before continuing.
{% endhint %}



:fire: **4.4. Configure port forwarding and/or firewall**



Specific to your networking setup or cloud provider settings, ensure your validator's firewall ports are open and reachable.



* **Lighthouse consensus client** requires port 9000 for tcp and udp
* **Execution client** requires port 30303 for tcp and udp



:chains: **4.5. Start the beacon chain**



Create a **systemd unit file** to define your`beacon-chain.service` configuration.

```
sudo nano /etc/systemd/system/beacon-chain.service
```



Paste the following configuration into the file.

```bash
# The eth beacon chain service (part of systemd)
# file: /etc/systemd/system/beacon-chain.service

[Unit]
Description=eth beacon chain service
Wants=network-online.target
After=network-online.target
Documentation=https://www.coincashew.com

[Service]
Type=simple
User=ethereum
Restart=on-failure
ExecStart=/home/ethereum/.cargo/bin/lighthouse bn \
  --network mainnet \
  --staking \
  --validator-monitor-auto \
  --metrics \
  --checkpoint-sync-url=https://beaconstate.info \
  --execution-endpoint http://127.0.0.1:8551 \
  --execution-jwt /secrets/jwtsecret

[Install]
WantedBy=multi-user.target
```



{% hint style="info" %}
**Checkpoint Sync**: allows consensus layer to start within minutes instead of days.

* Refer to [https://eth-clients.github.io/checkpoint-sync-endpoints/](https://eth-clients.github.io/checkpoint-sync-endpoints/) and feel free to pick one of the random `state` providers from the list, instead of the above endpoint`https://beaconstate.info`
* Do not trust any single checkpoint provider. Verify the **state root and block root** against multiple checkpoints to ensure you're on the correct chain.
{% endhint %}



To exit and save, press `Ctrl` + `X`, then `Y`, then`Enter`.



Update file permissions.

```bash
sudo chmod 644 /etc/systemd/system/beacon-chain.service
```



Run the following to enable auto-start at boot time and then start your beacon node service.

```
sudo systemctl daemon-reload
sudo systemctl enable beacon-chain
sudo systemctl start beacon-chain
```



{% hint style="success" %}
Nice work. Your beacon chain is now managed by the reliability and robustness of systemd.
{% endhint %}



:dna: **4.6. Start the validator**



Create a **systemd unit file** to define your `validator.service` configuration.

```
sudo nano /etc/systemd/system/validator.service
```



Paste the following configuration into the file.

```bash
# The eth validator service (part of systemd)
# file: /etc/systemd/system/validator.service

[Unit]
Description=eth validator service
Wants=network-online.target beacon-chain.service
After=network-online.target
Documentation=https://www.coincashew.com

[Service]
Type=simple
User=ethereum
Restart=on-failure
ExecStart=/home/ethereum/.cargo/bin/lighthouse vc \
 --network mainnet \
 --metrics \
 --suggested-fee-recipient 0x_CHANGE_THIS_TO_MY_ETH_FEE_RECIPIENT_ADDRESS

[Install]
WantedBy=multi-user.target
```

* Replace`0x_CHANGE_THIS_TO_MY_ETH_FEE_RECIPIENT_ADDRESS` with your own Ethereum address that you control. Tips are sent to this address and are immediately spendable, unlike the validator's attestation and block proposal rewards.



To exit and save, press `Ctrl` + `X`, then `Y`, then`Enter`.



Update file permissions.

```bash
sudo chmod 644 /etc/systemd/system/validator.service
```



Run the following to enable auto-start at boot time and then start your validator.

```
sudo systemctl daemon-reload
sudo systemctl enable validator
sudo systemctl start validator
```



{% hint style="success" %}
Nice work. Your validator is now managed by the reliability and robustness of systemd.
{% endhint %}
{% endtab %}

{% tab title="Nimbus" %}
{% hint style="info" %}
[Nimbus](https://our.status.im/tag/nimbus/) is a research project and a client implementation for Ethereum 2.0 designed to perform well on embedded systems and personal mobile devices, including older smartphones with resource-restricted hardware. The Nimbus team are from [Status](https://status.im/about/) the company best known for [their messaging app/wallet/Web3 browser](https://status.im) by the same name. Nimbus (Apache 2) is written in Nim, a language with Python-like syntax that compiles to C.
{% endhint %}

{% hint style="info" %}
**Note**: Nimbus combines both **validator client** and **beacon chain client** into one process.
{% endhint %}

:gear: **4.1. Build Nimbus from source**



Install dependencies.

```
sudo apt-get update
sudo apt-get install curl build-essential git -y
```



Install and build Nimbus.

```bash
mkdir ~/git
cd ~/git
git clone -b stable https://github.com/status-im/nimbus-eth2
cd nimbus-eth2
make update
make -j$(nproc) nimbus_beacon_node
```



{% hint style="info" %}
The build process may take a few minutes.
{% endhint %}



Verify Nimbus was installed properly by displaying the version.

```bash
cd $HOME/git/nimbus-eth2/build
./nimbus_beacon_node --version
```



Copy the binary file to `/usr/bin`

```bash
sudo cp $HOME/git/nimbus-eth2/build/nimbus_beacon_node /usr/bin
```



:tophat: **4.2. Import validator key**



Create a directory structure to store nimbus data.

```bash
sudo mkdir -p /var/lib/nimbus
```



Take ownership of this directory and set the correct permission level.

```bash
sudo chown ethereum:ethereum /var/lib/nimbus
sudo chmod 700 /var/lib/nimbus
```



The following command will import your validator keys.



Enter your **keystore password** to import accounts.

```bash
cd $HOME/git/nimbus-eth2
build/nimbus_beacon_node deposits import --data-dir=/var/lib/nimbus $HOME/staking-deposit-cli/validator_keys
```



Now you can verify the accounts were imported successfully by doing a directory listing.

```bash
ll /var/lib/nimbus/validators
```



You should see a folder named for each of your validator's pubkey.



{% hint style="danger" %}
WARNING: Do not import your validator keys into multiple validator clients and run them at the same time, or you might get slashed. If moving validators to a new setup or different validator client, ensure deletion of the previous validator keys before continuing.
{% endhint %}



:fire: **4.3. Configure port forwarding and/or firewall**



Specific to your networking setup or cloud provider settings, ensure your validator's firewall ports are open and reachable.



* **Nimbus consensus client** will use port 9000 for tcp and udp
* **Execution client** requires port 30303 for tcp and udp



:snowboarder: **4.4. Start the beacon chain and validator**



{% hint style="warning" %}
**Reminder**: Nimbus combines both the beacon chain and validator into one process.
{% endhint %}



**Running Checkpoint Sync**



{% hint style="info" %}
**Checkpoint Sync**: allows consensus layer to start within minutes instead of days.

* Refer to [https://eth-clients.github.io/checkpoint-sync-endpoints/](https://eth-clients.github.io/checkpoint-sync-endpoints/) and feel free to pick one of the random `state` providers from the list, instead of the endpoint`https://beaconstate.info`
* Do not trust any single checkpoint provider. Verify the **state root and block root** against multiple checkpoints to ensure you're on the correct chain.
{% endhint %}



Run the following command.

<pre><code>/usr/bin/nimbus_beacon_node trustedNodeSync \
--network=mainnet  \
--trusted-node-url=https://beaconstate.info \
<strong>--data-dir=/var/lib/nimbus \
</strong>--network=mainnet \
--backfill=false
</code></pre>



When the checkpoint sync is complete, you'll see the following message:

> Done, your beacon node is ready to serve you! Don't forget to check that you're on the canonical chain by comparing the checkpoint root with other online sources. See https://nimbus.guide/trusted-node-sync.html for more information.



**🛠 Setup systemd service**



Create a **systemd unit file** to define your`beacon-chain.service` configuration.

```
sudo nano /etc/systemd/system/beacon-chain.service
```



Paste the following configuration into the file.

```bash
# The eth beacon chain service (part of systemd)
# file: /etc/systemd/system/beacon-chain.service

[Unit]
Description=eth consensus layer beacon chain service
Wants=network-online.target
After=network-online.target
Documentation=https://www.coincashew.com

[Service]
Type=simple
User=ethereum
Restart=on-failure
ExecStart=/bin/bash -c '/usr/bin/nimbus_beacon_node \
  --network=mainnet \
  --data-dir=/var/lib/nimbus \
  --web3-url=ws://127.0.0.1:8551 \
  --metrics \
  --metrics-port=8008 \
  --suggested-fee-recipient=0x_CHANGE_THIS_TO_MY_ETH_FEE_RECIPIENT_ADDRESS \
  --jwt-secret="/secrets/jwtsecret"'

[Install]
WantedBy=multi-user.target
```

* Replace`0x_CHANGE_THIS_TO_MY_ETH_FEE_RECIPIENT_ADDRESS` with your own Ethereum address that you control. Tips are sent to this address and are immediately spendable, unlike the validator's attestation and block proposal rewards.



To exit and save, press `Ctrl` + `X`, then `Y`, then`Enter`.



Update file permissions.

```bash
sudo chmod 644 /etc/systemd/system/beacon-chain.service
```



Run the following to enable auto-start at boot time and then start your beacon node service.

```
sudo systemctl daemon-reload
sudo systemctl enable beacon-chain
sudo systemctl start beacon-chain
```



{% hint style="success" %}
Nice work. Your beacon chain is now managed by the reliability and robustness of systemd.
{% endhint %}
{% endtab %}

{% tab title="Teku" %}
{% hint style="info" %}
[PegaSys Teku](https://consensys.net/knowledge-base/ethereum-2/teku/) (formerly known as Artemis) is a Java-based Ethereum 2.0 client designed & built to meet institutional needs and security requirements. PegaSys is an arm of [ConsenSys](https://consensys.net) dedicated to building enterprise-ready clients and tools for interacting with the core Ethereum platform. Teku is Apache 2 licensed and written in Java, a language notable for its maturity & ubiquity.
{% endhint %}

{% hint style="info" %}
**Note**: Teku combines both **validator client** and **beacon chain client** into one process.
{% endhint %}

:gear: **4.1 Build Teku from source**



Install git.

```
sudo apt-get install git -y
```



Install Java 17 LTS.

```
sudo apt update
sudo apt install openjdk-17-jdk -y
```



Verify Java 17+ is installed.

```bash
java --version
```



Install and build Teku.

```bash
mkdir ~/git
cd ~/git
git clone https://github.com/ConsenSys/teku.git
cd teku
RELEASETAG=$(curl -s https://api.github.com/repos/ConsenSys/teku/releases/latest | jq -r .tag_name)
git checkout tags/$RELEASETAG
./gradlew distTar installDist
```



{% hint style="info" %}
This build process may take a few minutes.
{% endhint %}



Verify Teku was installed properly by displaying the version.

```bash
cd $HOME/git/teku/build/install/teku/bin
./teku --version
```



Copy the teku binary file to `/usr/bin/teku`

```bash
sudo cp -r $HOME/git/teku/build/install/teku /usr/bin/teku
```



:fire: **4.2. Configure port forwarding and/or firewall**



Specific to your networking setup or cloud provider settings, ensure your validator's firewall ports are open and reachable.



* **Teku consensus client** will use port 9000 for tcp and udp
* **Execution client** requires port 30303 for tcp and udp



:snowboarder: **4.3. Configure the beacon chain and validator**



{% hint style="info" %}
Teku combines both the beacon chain and validator into one process.
{% endhint %}



Setup a directory structure for Teku.

<pre class="language-bash"><code class="lang-bash">sudo mkdir -p /var/lib/teku
sudo mkdir -p /etc/teku
<strong>sudo chown ethereum:ethereum /var/lib/teku
</strong>sudo chown ethereum:ethereum /etc/teku
</code></pre>



Copy your `validator_files` directory to the data directory we created above.

```bash
cp -r $HOME/staking-deposit-cli/validator_keys /var/lib/teku
```



Remove the extra deposit\_data file. Answer 'y' to remove write-protected regular file.

```
rm /var/lib/teku/validator_keys/deposit_data*
```



{% hint style="danger" %}
WARNING: Do not import your validator keys into multiple validator clients and run them at the same time, or you might get slashed. If moving validators to a new setup or different validator client, ensure deletion of the previous validator keys before continuing.
{% endhint %}



Storing your **keystore password** in a text file is required so that Teku can decrypt and load your validators automatically.



Replace `<my_keystore_password_goes_here>` with your **keystore password** between the single quotation marks and then run the command to save it to validators-password.txt

```bash
echo '<my_keystore_password_goes_here>' > $HOME/validators-password.txt
```



Confirm that your **keystore password** is correct.

```bash
cat $HOME/validators-password.txt
```



Move the password file and make it read-only.

```bash
sudo mv $HOME/validators-password.txt /etc/teku/validators-password.txt
sudo chmod 600 /etc/teku/validators-password.txt
```



Clear the bash history in order to remove traces of keystore password.

```bash
shred -u ~/.bash_history && touch ~/.bash_history
```



Create your teku.yaml configuration file.

```bash
sudo nano /etc/teku/teku.yaml
```



Paste the following configuration into the file.

```bash
# network
network: "mainnet"
initial-state: "https://beaconstate.info/eth/v2/debug/beacon/states/finalized"

# validators
validator-keys: "/var/lib/teku/validator_keys:/var/lib/teku/validator_keys"

# execution engine
ee-endpoint: http://localhost:8551
ee-jwt-secret-file: "/secrets/jwtsecret"

# fee recipient
validators-proposer-default-fee-recipient: "<0x_CHANGE_THIS_TO_MY_ETH_FEE_RECIPIENT_ADDRESS>"

# metrics
metrics-enabled: true
metrics-port: 8008

# database
data-path: "/var/lib/teku"
data-storage-mode: "prune"
```



{% hint style="info" %}
**Checkpoint Sync**: allows consensus layer to start within minutes instead of days.

* Refer to [https://eth-clients.github.io/checkpoint-sync-endpoints/](https://eth-clients.github.io/checkpoint-sync-endpoints/) and feel free to pick one of the random `state` providers from the list, instead of the above endpoint`https://beaconstate.info`
* Do not trust any single checkpoint provider. Verify the **state root and block root** against multiple checkpoints to ensure you're on the correct chain.
{% endhint %}

* Replace`<0x_CHANGE_THIS_TO_MY_ETH_FEE_RECIPIENT_ADDRESS>` with your own Ethereum address that you control. Tips are sent to this address and are immediately spendable, unlike the validator's attestation and block proposal rewards.



:tophat: **4.4 Import validator key**



{% hint style="info" %}
When specifying directories for your validator-keys, Teku expects to find identically named keystore and password files.

For example `keystore-m_12221_3600_1_0_0-11222333.json` and `keystore-m_12221_3600_1_0_0-11222333.txt`
{% endhint %}



Create a corresponding password file for every one of your validators.

```bash
for f in /var/lib/teku/validator_keys/keystore*.json; do cp /etc/teku/validators-password.txt /var/lib/teku/validator_keys/$(basename $f .json).txt; done
```



Verify that your validator's keystore and validator's passwords are present by checking the following directory.

```bash
ll /var/lib/teku/validator_keys
```

Update directory ownership.

```bash
sudo chown -R ethereum:ethereum /var/lib/teku
sudo chown -R ethereum:ethereum /etc/teku
```

:checkered\_flag: **4.5. Start the beacon chain and validator**



Use **systemd** to manage starting and stopping teku.

***

:tools: **Setup systemd service**

***

Run the following to create a **unit file** to define your`beacon-chain.service` configuration. Simply copy and paste.

```bash
cat > $HOME/beacon-chain.service << EOF
# The eth beacon chain service (part of systemd)
# file: /etc/systemd/system/beacon-chain.service

[Unit]
Description=eth consensus layer beacon chain service
Wants=network-online.target
After=network-online.target
Documentation=https://www.coincashew.com

[Service]
User=ethereum
ExecStart=/usr/bin/teku/bin/teku -c /etc/teku/teku.yaml
Restart=on-failure
Environment=JAVA_OPTS=-Xmx5g

[Install]
WantedBy=multi-user.target
EOF
```



Move the unit file to `/etc/systemd/system`

```bash
sudo mv $HOME/beacon-chain.service /etc/systemd/system/beacon-chain.service
```



Update file permissions.

```bash
sudo chmod 644 /etc/systemd/system/beacon-chain.service
```



Run the following to enable auto-start at boot time and then start your beacon node service.

```
sudo systemctl daemon-reload
sudo systemctl enable beacon-chain
sudo systemctl start beacon-chain
```



{% hint style="success" %}
Nice work. Your beacon chain is now managed by the reliability and robustness of systemd.
{% endhint %}
{% endtab %}

{% tab title="Prysm" %}
{% hint style="info" %}
[Prysm](https://github.com/prysmaticlabs/prysm) is a Go implementation of Ethereum 2.0 protocol with a focus on usability, security, and reliability. Prysm is developed by [Prysmatic Labs](https://prysmaticlabs.com), a company with the sole focus on the development of their client. Prysm is written in Go and released under a GPL-3.0 license.
{% endhint %}



:gear: **4.1. Install Prysm**

```bash
mkdir ~/prysm && cd ~/prysm
curl https://raw.githubusercontent.com/prysmaticlabs/prysm/master/prysm.sh --output prysm.sh && chmod +x prysm.sh
```



:fire: **4.2. Configure port forwarding and/or firewall**



Specific to your networking setup or cloud provider settings, ensure your validator's firewall ports are open and reachable.



* **Prysm consensus client** will use port 12000 for udp and port 13000 for tcp
* **Execution client** requires port 30303 for tcp and udp



:tophat: **4.3. Import validator key**

***

```bash
$HOME/prysm/prysm.sh validator accounts import --mainnet --keys-dir=$HOME/staking-deposit-cli/validator_keys
```

* Type "accept" to accept terms of use
* Press enter to accept default wallet location
* Enter a new **prysm-only password** to encrypt your local prysm wallet files
* and enter the **keystore password** for your imported accounts.



{% hint style="info" %}
For simplicity, use the same password for the **keystore** and **prysm-only password**.
{% endhint %}



Verify your validators imported successfully.

```bash
$HOME/prysm/prysm.sh validator accounts list --mainnet
```



Confirm your validator's pubkeys are listed.

> \#Example output:
>
> Showing 1 validator account View the eth1 deposit transaction data for your accounts by running \`validator accounts list --show-deposit-data
>
> Account 0 | pens-brother-heat\
> \[validating public key] 0x2374.....7121



{% hint style="danger" %}
WARNING: Do not import your validator keys into multiple validator clients and run them at the same time, or you might get slashed. If moving validators to a new setup or different validator client, ensure deletion of the previous validator keys before continuing.
{% endhint %}



:snowboarder: **4.4. Start the beacon chain**



:tools: **Setup systemd service**



Create a **systemd unit file** to define your`beacon-chain.service` configuration.

```
sudo nano /etc/systemd/system/beacon-chain.service
```



Paste the following configuration into the file.

```bash
# The eth beacon chain service (part of systemd)
# file: /etc/systemd/system/beacon-chain.service

[Unit]
Description=eth consensus layer beacon chain service
Wants=network-online.target
After=network-online.target
Documentation=https://www.coincashew.com

[Service]
Type=simple
User=ethereum
Restart=on-failure
ExecStart=/home/ethereum/prysm/prysm.sh beacon-chain \
  --mainnet \
  --checkpoint-sync-url=https://beaconstate.info \
  --genesis-beacon-api-url=https://beaconstate.info \
  --execution-endpoint=http://localhost:8551 \
  --jwt-secret=/secrets/jwtsecret \
  --suggested-fee-recipient=0x_CHANGE_THIS_TO_MY_ETH_FEE_RECIPIENT_ADDRESS \
  --accept-terms-of-use

[Install]
WantedBy=multi-user.target
```



{% hint style="info" %}
**Checkpoint Sync**: allows consensus layer to start within minutes instead of days.

* Refer to [https://eth-clients.github.io/checkpoint-sync-endpoints/](https://eth-clients.github.io/checkpoint-sync-endpoints/) and feel free to pick one of the random `state` providers from the list, instead of the above endpoint`https://beaconstate.info`
* Do not trust any single checkpoint provider. Verify the **state root and block root** against multiple checkpoints to ensure you're on the correct chain.
{% endhint %}

* Replace`0x_CHANGE_THIS_TO_MY_ETH_FEE_RECIPIENT_ADDRESS` with your own Ethereum address that you control. Tips are sent to this address and are immediately spendable, unlike the validator's attestation and block proposal rewards.



To exit and save, press `Ctrl` + `X`, then `Y`, then`Enter`.



Update file permissions.

```bash
sudo chmod 644 /etc/systemd/system/beacon-chain.service
```



Run the following to enable auto-start at boot time and then start your beacon node service.

```
sudo systemctl daemon-reload
sudo systemctl enable beacon-chain
sudo systemctl start beacon-chain
```



{% hint style="success" %}
Nice work. Your beacon chain is now managed by the reliability and robustness of systemd.
{% endhint %}



:dna: **4.5. Start the validator**



Store your **prysm-only password** in a file and make it read-only.

This is required so that Prysm can decrypt and load your validators.



Replace **`<my_password_goes_here>`** with your **prysm-only** password.

```bash
echo '<my_password_goes_here>' > /home/ethereum/.eth2validators/validators-password.txt
sudo chmod 600 /home/ethereum/.eth2validators/validators-password.txt
```



Verify your password is correct.

```
cat $HOME/.eth2validators/validators-password.txt
```



Clear the bash history in order to remove traces of your **prysm-only password.**

```bash
shred -u ~/.bash_history && touch ~/.bash_history
```



:tools: **Setup systemd service**

***

Create a **systemd unit file** to define your `validator.service` configuration.

```
sudo nano /etc/systemd/system/validator.service
```



Paste the following configuration into the file.

```bash
# The eth validator service (part of systemd)
# file: /etc/systemd/system/validator.service

[Unit]
Description=eth validator service
Wants=network-online.target beacon-chain.service
After=network-online.target
Documentation=https://www.coincashew.com

[Service]
Type=simple
User=ethereum
Restart=on-failure
ExecStart=/home/ethereum/prysm/prysm.sh validator \
  --mainnet \
  --accept-terms-of-use \
  --wallet-password-file /home/ethereum/.eth2validators/validators-password.txt \
  --suggested-fee-recipient 0x_CHANGE_THIS_TO_MY_ETH_FEE_RECIPIENT_ADDRESS

[Install]
WantedBy=multi-user.target
```

* Replace`0x_CHANGE_THIS_TO_MY_ETH_FEE_RECIPIENT_ADDRESS` with your own Ethereum address that you control. Tips are sent to this address and are immediately spendable, unlike the validator's attestation and block proposal rewards.



To exit and save, press `Ctrl` + `X`, then `Y`, then`Enter`.



Update file permissions.

```bash
sudo chmod 644 /etc/systemd/system/validator.service
```



Run the following to enable auto-start at boot time and then start your validator.

```
sudo systemctl daemon-reload
sudo systemctl enable validator
sudo systemctl start validator
```
{% endtab %}

{% tab title="Lodestar" %}
{% hint style="info" %}
[Lodestar ](https://lodestar.chainsafe.io)**is a Typescript implementation** of the official [Ethereum 2.0 specification](https://github.com/ethereum/eth2.0-specs) by the [ChainSafe.io](https://lodestar.chainsafe.io) team. In addition to the beacon chain client, the team is also working on 22 packages and libraries. A complete list can be found [here](https://hackmd.io/CcsWTnvRS\_eiLUajr3gi9g). Finally, the Lodestar team is leading the Eth2 space in light client research and development and has received funding from the EF and Moloch DAO for this purpose.
{% endhint %}



:gear: **4.1 Build Lodestar from source**



Install curl and git.

```bash
sudo apt-get install gcc g++ make git curl -y
```



Install yarn.

```bash
curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
sudo apt update
sudo apt install yarn -y
```



Confirm yarn is installed properly.

```bash
yarn --version
# Should output version >= 1.22.4
```



Install nodejs.

```
curl -sL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs
```



Install and build Lodestar.

```bash
mkdir ~/git
cd ~/git
git clone -b stable https://github.com/chainsafe/lodestar.git
cd lodestar
yarn install --ignore-optional
yarn run build
```



{% hint style="info" %}
This build process may take a few minutes.
{% endhint %}



Verify Lodestar was installed properly by displaying the version.

```
./lodestar --version
```



Setup a directory structure for Lodestar.

```bash
sudo mkdir -p /var/lib/lodestar
sudo chown ethereum:ethereum /var/lib/lodestar
```



:fire: **4.2. Configure port forwarding and/or firewall**



Specific to your networking setup or cloud provider settings, ensure your validator's firewall ports are open and reachable.



* **Lodestar consensus client** will use port 9000
* **Execution client** requires port 30303



:tophat: **4.3. Import validator key**

```bash
./lodestar validator import \
  --network mainnet \
  --dataDir /var/lib/lodestar \
  --keystore $HOME/staking-deposit-cli/validator_keys
```



Enter your **keystore password** to import accounts.



Confirm your keys were imported properly.

```
./lodestar validator list \
  --network mainnet \
  --dataDir /var/lib/lodestar
```



{% hint style="danger" %}
WARNING: Do not import your validator keys into multiple validator clients and run them at the same time, or you might get slashed. If moving validators to a new setup or different validator client, ensure deletion of the previous validator keys before continuing.
{% endhint %}



:snowboarder: **4.4. Start the beacon chain and validator**



Run the beacon chain automatically with systemd.

***

**🛠 Setup systemd service**



Create a **systemd unit file** to define your`beacon-chain.service` configuration.

```
sudo nano /etc/systemd/system/beacon-chain.service
```



Paste the following configuration into the file.

```bash
# The eth2 beacon chain service (part of systemd)
# file: /etc/systemd/system/beacon-chain.service

[Unit]
Description=eth2 beacon chain service
Wants=network-online.target
After=network-online.target
Documentation=https://www.coincashew.com

[Service]
Type=simple
User=ethereum
Restart=on-failure
WorkingDirectory=/home/ethereum/git/lodestar
ExecStart=/home/ethereum/git/lodestar/lodestar beacon \
  --network mainnet \
  --dataDir /var/lib/lodestar \
  --metrics true \
  --checkpointSyncUrl https://beaconstate.info \
  --jwt-secret /secrets/jwtsecret \
  --execution.urls http://127.0.0.1:8551 \
  --suggestedFeeRecipient 0x_CHANGE_THIS_TO_MY_ETH_FEE_RECIPIENT_ADDRESS

[Install]
WantedBy=multi-user.target
```



{% hint style="info" %}
**Checkpoint Sync**: allows consensus layer to start within minutes instead of days.

* Refer to [https://eth-clients.github.io/checkpoint-sync-endpoints/](https://eth-clients.github.io/checkpoint-sync-endpoints/) and feel free to pick one of the random `state` providers from the list, instead of the above endpoint`https://beaconstate.info`
* Do not trust any single checkpoint provider. Verify the **state root and block root** against multiple checkpoints to ensure you're on the correct chain.
{% endhint %}

* Replac**e**`0x_CHANGE_THIS_TO_MY_ETH_FEE_RECIPIENT_ADDRESS` with your own Ethereum address that you control. Tips are sent to this address and are immediately spendable, unlike the validator's attestation and block proposal rewards.



To exit and save, press `Ctrl` + `X`, then `Y`, then`Enter`.



Update file permissions.

```bash
sudo chmod 644 /etc/systemd/system/beacon-chain.service
```



Run the following to enable auto-start at boot time and then start your beacon node service.

```
sudo systemctl daemon-reload
sudo systemctl enable beacon-chain
sudo systemctl start beacon-chain
```



{% hint style="success" %}
Nice work. Your beacon chain is now managed by the reliability and robustness of systemd.
{% endhint %}



:dna: **4.5. Start the validator**



:tools: **Setup systemd service**



Create a **systemd unit file** to define your `validator.service` configuration.

```
sudo nano /etc/systemd/system/validator.service
```



Paste the following configuration into the file.

```bash
# The eth2 validator service (part of systemd)
# file: /etc/systemd/system/validator.service

[Unit]
Description=eth2 validator service
Wants=network-online.target beacon-chain.service
After=network-online.target
Documentation=https://www.coincashew.com

[Service]
Type=simple
User=ethereum
Restart=on-failure
WorkingDirectory=/home/ethereum/git/lodestar
ExecStart=/home/ethereum/git/lodestar/lodestar validator \
  --network mainnet \
  --dataDir /var/lib/lodestar \
  --suggestedFeeRecipient 0x_CHANGE_THIS_TO_MY_ETH_FEE_RECIPIENT_ADDRESS

[Install]
WantedBy=multi-user.target
```



* Replace`0x_CHANGE_THIS_TO_MY_ETH_FEE_RECIPIENT_ADDRESS` with your own Ethereum address that you control. Tips are sent to this address and are immediately spendable, unlike the validator's attestation and block proposal rewards.



To exit and save, press `Ctrl` + `X`, then `Y`, then`Enter`.



Update file permissions.

```bash
sudo chmod 644 /etc/systemd/system/validator.service
```



Run the following to enable auto-start at boot time and then start your validator.

```
sudo systemctl daemon-reload
sudo systemctl enable validator
sudo systemctl start validator
```



{% hint style="success" %}
Nice work. Your validator is now managed by the reliability and robustness of systemd.
{% endhint %}
{% endtab %}
{% endtabs %}

## :tools: H**elpful Consensus Client systemd commands**

{% tabs %}
{% tab title="beacon-chain" %}
**🗄 Viewing and filtering logs**

```bash
#view and follow the log
journalctl --unit=beacon-chain -f
```

```bash
#view log since yesterday
journalctl --unit=beacon-chain --since=yesterday
```

```bash
#view log since today
journalctl --unit=beacon-chain --since=today
```

```bash
#view log between a date
journalctl --unit=beacon-chain --since='2020-12-01 00:00:00' --until='2020-12-02 12:00:00'
```



:mag\_right: **View the status of the beacon chain**

```
sudo systemctl status beacon-chain
```



:arrows\_counterclockwise: **Restarting the beacon chain**

```
sudo systemctl reload-or-restart beacon-chain
```



:octagonal\_sign: **Stopping the beacon chain**

```
sudo systemctl stop beacon-chain
```
{% endtab %}

{% tab title="validator" %}
**🗄 Viewing and filtering logs**

```bash
#view and follow the log
journalctl --unit=validator -f
```

```bash
#view log since yesterday
journalctl --unit=validator --since=yesterday
```

```bash
#view log since today
journalctl --unit=validator --since=today
```

```bash
#view log between a date
journalctl --unit=validator --since='2020-12-01 00:00:00' --until='2020-12-02 12:00:00'
```



:mag\_right: **View the status of the validator**

```
sudo systemctl status validator
```



:arrows\_counterclockwise: **Restarting the validator**

```
sudo systemctl reload-or-restart validator
```



:octagonal\_sign: **Stopping the validator**

```
sudo systemctl stop validator
```
{% endtab %}
{% endtabs %}

## :track\_next: Next Steps

{% hint style="info" %}
**Sync Timeline**: Syncing the consensus client is instantaneous with checkpoint sync but the execution client can take up to 1 week. On high-end machines with gigabit internet, expect your node to be fully syncing to take less than a day.
{% endhint %}

{% hint style="warning" %}
**Patience required**: If you're checking the logs and see any warnings or errors, please be patient as these will normally resolve once both your execution and consensus clients are fully synced to the Ethereum network.



**How do I know I'm fully synced?**

* Check your execution client's logs and compare the block number against the most recent block on [etherscan.io](https://etherscan.io/)
  * Check EL logs: `journalctl -fu eth1`
* Check your consensus client's logs and compare the slot number against the most recent slot on [beaconcha.in](https://beaconcha.in/)
  * Check CL logs: `journalctl -fu beacon-chain`
{% endhint %}

{% hint style="info" %}
**Activation Queue**: Once your EL+CL is synced, validator up and running, you just wait for activation. This process can take 24+ hours. Only 900 new validators can join per day. Check the queue length: [https://wenmerge.com ](https://wenmerge.com)

**Activated**: When you're activated, your validator will begin creating and voting on blocks while earning staking rewards.

**Quick monitoring**: Use [https://beaconcha.in/](https://beaconcha.in) to create alerts and track your validator's performance.
{% endhint %}

{% hint style="success" %}
:tada: Congrats! You've finished the primary steps of setting up your validator. You're now an Ethereum staker!
{% endhint %}

### :thumbsup: Recommended Steps

* Subscribe to your Execution Client and Consensus Client's Github repository to be notified of new releases. Hit the Notifications button.
* Join the [community on Discord and Reddit](../../../guide-or-how-to-setup-a-validator-on-eth2-mainnet/joining-the-community-on-discord-and-reddit.md#discord) to discuss all things staking related.
* Familiarize yourself with [Part II - Maintenance](../../../guide-or-how-to-setup-a-validator-on-eth2-mainnet/part-ii-maintenance/) section, as you'll need to keep your staking node running at its best.
* Up your staking understanding with the [EthStaker Knowledge Base](https://docs.ethstaker.cc/ethstaker-knowledge-base/)

### :checkered\_flag: Optional Steps

* Setup [Monitoring with Grafana and Prometheus](../../../guide-or-how-to-setup-a-validator-on-eth2-mainnet/part-i-installation/monitoring-your-validator-with-grafana-and-prometheus.md)
* Setup [Mobile App Notifications and Monitoring by beaconcha.in](../../../guide-or-how-to-setup-a-validator-on-eth2-mainnet/part-i-installation/mobile-app-node-monitoring-by-beaconchain.md)
* Setup [External Monitoring with Uptime Check by Google Cloud](../../../guide-or-how-to-setup-a-validator-on-eth2-mainnet/part-i-installation/monitoring-with-uptime-check-by-google-cloud.md)
* Setup [MEV-boost](../../../mev-boost/) for extra staking rewards!
* Familiarize yourself with [Part III - Tips](../../../guide-or-how-to-setup-a-validator-on-eth2-mainnet/part-iii-tips/) section, as you dive deeper into staking.

### :telephone: **Need extra live support?**

* Find Ethstaker frens on the [Ethstaker](https://discord.io/ethstaker) Discord and [coincashew](https://discord.gg/w8Bx8W2HPW) Discord.
* Use reddit: [r/Ethstaker](https://www.reddit.com/r/ethstaker/), or [DMs](https://www.reddit.com/user/coincashew), or [r/coincashew](https://www.reddit.com/r/coincashew/)

### :heart\_decoration: Like these guides?

* **Audience-funded guide**: If you found this helpful, [please consider supporting it directly.](../../../../../donations.md) :pray:
* **Support us on Gitcoin Grants:** We build this guide exclusively by community support!
* **Feedback or pull-requests**: [https://github.com/coincashew/coincashew](https://github.com/coincashew/coincashew)

## Last Words

> I stand upon the shoulders of giants and as such, invite you to stand upon mine. Use my work with or without attribution; I make no claim of "intellectual property." My ideas are the result of countless millenia of evolution - they belong to humanity.

<figure><img src="../../../../../.gitbook/assets/leslie-solo.png" alt=""><figcaption><p>This is Leslie, the official mascot of Eth Staking</p></figcaption></figure>
