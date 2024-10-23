# Verifying Your Mnemonic Phrase

Using the staking-deposit-cli tool, ensure you can regenerate the same eth2 key pairs by restoring your `validator_keys`

```bash
cd $HOME/staking-deposit-cli 
./deposit.sh existing-mnemonic --chain mainnet
```

{% hint style="info" %}
When the **pubkey** in both **keystore** files are **identical**, this means your mnemonic phrase is veritably correct. Other fields will be different because of salting.
{% endhint %}
