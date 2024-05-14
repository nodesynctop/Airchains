# Airchains
This manual offers a detailed, step-by-step approach for establishing and operating a full validator node on the Junction.

Website: https://www.airchains.io

Twitter: https://twitter.com/airchains_io

Discord: https://discord.gg/airchains

#### Faucet on Discord

#### Services

Explorer: https://explorer.nodesync.top/Junction-Testnet

API: https://junction-testnet-api.nodesync.top

RPC: https://junction-testnet-rpc.nodesync.top
# 1. Minimum hardware requirement
2 Cores, 4G Ram,  40G SSD, Ubuntu 22.04
# 2. Auto Install
Use snapshot:
```
sudo apt install curl -y && wget https://nodesync.top/Testnet/Junction_auto && chmod +x Junction_auto && ./Junction_auto
```
# 2.1 Wallet
Add New Wallet Key - Save seed
```
junctiond keys add wallet
```
Recover existing key
```
junctiond keys add wallet --recover
```
List All Keys
```
junctiond keys list
```
# 2.2 Query Wallet Balance
```
junctiond q bank balances $(junctiond keys show wallet -a)
```
# 2.3 Check sync status
#### False is synced
```
junctiond status 2>&1 | jq .sync_info
```
# 2.4 Create Validator
#### Obtain your validator public key by running the following command:
```
junctiond comet show-validator
```
#### The output will be similar to this (with a different key):

{"@type":"/cosmos.crypto.ed25519.PubKey","key":"lR1d7YBVK5jYijOfWVKRFoWCsS4dg3kagT7LB9GnG8I="}
#### Then, create a file named `validator.json` with the following content:
```
nano $HOME/validator.json
```
#### Change your info, from "pubkey" to "details"  and Save them
```
{
	"pubkey": {"@type":"/cosmos.crypto.ed25519.PubKey","key":"lR1d7YBVK5jYijOfWVKRFoWCsS4dg3kagT7LB9GnG8I="},
	"amount": "1000000amf",
	"moniker": "<validator-name>",
	"identity": "optional identity signature (ex. UPort or Keybase)",
	"website": "validator's (optional) website",
	"security": "validator's (optional) security contact email",
	"details": "validator's (optional) details",
	"commission-rate": "0.1",
	"commission-max-rate": "0.2",
	"commission-max-change-rate": "0.01",
	"min-self-delegation": "1"
}
```
#### Finally, we're ready to submit the transaction to create the validator:
```
junctiond tx staking create-validator validator.json --from wallet --chain-id junction --fees 5000amf -y
```
# 2.5 Delegate Token to your own validator
```
junctiond tx staking delegate $(junctiond keys show wallet --bech val -a)  1000000amf \
--from wallet --chain-id junction \
--fees 5000amf \
-y
```
# 2.6 Withdraw rewards and commission from your validator
```
junctiond tx distribution withdraw-rewards $(junctiond keys show wallet --bech val -a) \
--from wallet --chain-id junction \
--fees 5000amf \
-y
```
# 2.7 Unjail validator
```
junctiond tx slashing unjail --from wallet --chain-id junction --fees 5000amf -y
```
# 2.8 Services Management
```
# Reload Service
sudo systemctl daemon-reload

# Enable Service
sudo systemctl enable junctiond

# Disable Service
sudo systemctl disable junctiond

# Start Service
sudo systemctl start junctiond

# Stop Service
sudo systemctl stop junctiond

# Restart Service
sudo systemctl restart junctiond

# Check Service Status
sudo systemctl status junctiond

# Check Service Logs
sudo journalctl -u junctiond -f --no-hostname -o cat
```
# 3. Backup Validator
#### Important
```
cat $HOME/.junction/config/priv_validator_key.json
```
# 4. Remove node
```
sudo systemctl stop junctiond
sudo systemctl disable junctiond
rm -rf /etc/systemd/system/junctiond.service
sudo systemctl daemon-reload
sudo rm -f /usr/local/bin/junctiond
sudo rm -f $(which junctiond)
sudo rm -rf $HOME/.junction
sudo rm -rf Junction_auto
```
# 5. Snapshot
```
sudo systemctl stop junctiond
cp $HOME/.junction/data/priv_validator_state.json $HOME/.junction/priv_validator_state.json.backup
rm -rf $HOME/.junction/data $HOME/.junction/wasmPath

curl https://files.nodesync.top/Junction/junction.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME/.junction

mv $HOME/.junction/priv_validator_state.json.backup $HOME/.junction/data/priv_validator_state.json

sudo systemctl restart junctiond && sudo journalctl -u junctiond -f --no-hostname -o cat
```


