# Stride node setup - STRIDE-TESTNET-2
Official documentation:

> Validator setup instructions:



> Explorer:

https://teritori.explorers.guru/

> Network Chain ID

teritori-testnet-v2

> Denom: 

utori

## Hardware Requirements

Like any Cosmos-SDK chain, the hardware requirements are pretty easy.

`Recommended Hardware Requirements`

- 4x CPUs; the faster clock speed the better
- 8GB RAM
- 200GB of storage (SSD or NVME)
- Permanent Internet connection (traffic will be minimal during mainnet; 10Mbps will be plenty - for production at least 100Mbps is expected)

## Preparing the server

Update the repositories
```
sudo apt update && sudo apt upgrade -y
```
Install necessary utilities
```
sudo apt install curl build-essential git wget jq make gcc tmux htop nvme-cli pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip libleveldb-dev -y
```
Full test
```
bash <(wget -qO- -o /dev/null yabs.sh)
```
Test disks only
```
bash <(wget -qO- -o /dev/null yabs.sh) -ig
```
Test internet speed only
```
bash <(wget -qO- -o /dev/null yabs.sh) -dg
```
Test the system performance only
```
bash <(wget -qO- -o /dev/null yabs.sh) -di
```

**Installing GO (One command)**

```
ver="1.18.1" && \
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
rm "go$ver.linux-amd64.tar.gz" && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile && \
source $HOME/.bash_profile && \
go version
```

## Node deployment

:heavy_exclamation_mark:IMPORTANT - in the commands below, change everything in <> to your value and remove <>:heavy_exclamation_mark:

```
git clone https://github.com/TERITORI/teritori-chain && cd teritori-chain
git checkout teritori-testnet-v2
make install
```
Check version
```
teritorid version --long | head
```
Initialize a node to create the necessary configuration files
```
teritorid init <name_moniker> --chain-id teritori-testnet-v2
```
Download `genesis file`
```
wget -O $HOME/.teritorid/config/genesis.json "https://raw.githubusercontent.com/TERITORI/teritori-chain/teritori-testnet-v2/genesis/genesis.json"
```
Check version of a `genesis file`
```
sha256sum ~/.teritorid/config/genesis.json
```
Check the state of the validator blocks at the initial stage
```
cd && cat .teritorid/data/priv_validator_state.json
```
`{
  "height": "0",
  "round": 0,
  "step": 0
}`

If the values above are not equal to zero, then execute the command, but if they are equal to zero, then skip this step
```
teritorid tendermint unsafe-reset-all --home $HOME/.teritorid
```
Download the `addrbook file`
```

```
## Setting up the node configuration
Edit the config so that we no longer use the `chain-id` flag for each CLI command in client.toml
```
teritorid config chain-id teritori-testnet-v2
```
If necessary, configure the keyring-backend in `client.toml`
```
teritorid config keyring-backend os
```
Setting the minimum price for gas in `app.toml`
```
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0025ustrd\"/;" ~/teritorid/config/app.toml
```
Add seeds/bpeers/peers to `config.toml`
```
external_address=$(wget -qO- eth0.me)
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/teritorid/config/config.toml
```
```
peers="0dde2ae55624d822eeea57d1b5e1223b6019a531@176.9.149.15:26656,4d2ea61e6195ee4e449c1e6132cabce98f7d94e1@5.9.40.222:26656,bceb776975aab62bcfd501969c0e1a2734ed7c2e@176.9.19.162:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.teritorid/config/config.toml
```
```
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.teritorid/config/config.toml
```
Increase the number of incoming and outgoing peers for the connection, except for persistent peers in `config.toml`
```
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 100/g' $HOME/.teritorid/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 100/g' $HOME/.teritorid/config/config.toml
```
**(Optional)** Configure pruning with one command `app.toml`
```
pruning="custom" && \
pruning_keep_recent="100" && \
pruning_keep_every="0" && \
pruning_interval="50" && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.teritorid/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.teritorid/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.teritorid/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.teritorid/config/app.toml
```
**(Optional)** Turn off indexing in `config.toml`
```
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.teritorid/config/config.toml
```
**(Optional)** On/off snapshots in `app.toml`
```
snapshot_interval=1000 && \
sed -i.bak -e "s/^snapshot-interval *=.*/snapshot-interval = \"$snapshot_interval\"/" ~/.teritorid/config/app.toml
```
By default snapshots are disabled `snapshot-interval=0`
## Create a service file
```
sudo tee /etc/systemd/system/teritorid.service > /dev/null <<EOF
[Unit]
Description=strided
After=network-online.target

[Service]
User=$USER
ExecStart=$(which teritorid) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
```
sudo systemctl daemon-reload && \
sudo systemctl enable teritorid && \
sudo systemctl restart teritorid && sudo journalctl -u teritorid -f -o cat
```
:heavy_exclamation_mark:If after start the node cannot connect to peers for a long time, then look for new peers or ask for addrbook.json in discord and so on:heavy_exclamation_mark:

Stop the node, delete the address book and reset node
```
sudo systemctl stop teritorid
rm $HOME/.teritorid/config/addrbook.json
strided tendermint unsafe-reset-all --home $HOME/.teritorid
```
Restart the node
```
sudo systemctl restart teritorid && journalctl -u teritorid -f -o cat
```
Create or restore the wallet and save the memnomics and other information as needed

:heavy_exclamation_mark:Don't forget to save yourself a memo, because you are responsible for your wallet:heavy_exclamation_mark:

Set up a wallet
```
teritorid keys add <name_wallet> --keyring-backend os
```
Recover a wallet
```
teritorid keys add <name_wallet> --recover --keyring-backend os
```
Connect ledger wallet
```
teritorid keys add <name_wallet> --ledger 
```
Create a validator

:heavy_exclamation_mark:Don't forget to save `priv_validator_key.json` 
```
teritorid tx staking create-validator \
--chain-id teritori-testnet-v2 \
--commission-rate 0.05 \
--commission-max-rate 0.2 \
--commission-max-change-rate 0.1 \
--min-self-delegation "1000000" \
--amount 1000000utori \
--pubkey $(teritorid tendermint show-validator) \
--moniker "<name_moniker>" \
--from <name_wallet> \
--fees 555utori
```
## Useful Commands

Check block heights
```
teritorid status 2>&1 | jq ."SyncInfo"."latest_block_height"
```
Check logs (all logs and the last 100 lines)
```
sudo journalctl -u teritorid -f -o cat
sudo journalctl --lines=100 --follow --unit teritorid
```
Check status
```
curl localhost:26657/status
```
Check a balance of the wallet
```
teritorid q bank balances <address>
```
Collect commissions + rewards
```
teritorid tx distribution withdraw-rewards <valoper_address> --from <name_wallet> --fees 500utori --commission -y
```
Delegate some more coins to yourself (for example, 1 coin is sent)
```
teritorid tx staking delegate <valoper_address> 1000000ustrd --from <name_wallet> --fees 500utori -y
```
Send coins to another address
```
teritorid tx bank send <name_wallet> <address> 1000000utori --fees 500utori -y
```
Get out of jail
```
teritorid tx slashing unjail --from <name_wallet> --fees 500utori -y
```
Display a list of wallets
```
teritorid keys list
```
Vote for the proposal 
```
teritorid tx gov vote 1 yes --from <name_wallet> --fees 555ustrd
```
Delete a node
```
sudo systemctl stop teritorid && \
sudo systemctl disable teritorid && \
rm /etc/systemd/system/teritorid.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf .teritorid teritori-chain && \
rm -rf $(which teritorid)
```
