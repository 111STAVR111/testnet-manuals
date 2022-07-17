# stride node setup - STRIDE-1
Official documentation:

> Validator setup instructions:

https://github.com/Stride-Labs/testnet

> Explorer:

https://poolparty.stride.zone/

> Network Chain ID

STRIDE-1

> Denom: 

ustrd

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
Check that the hard disks work
```
curl -sL yabs.sh | bash -s -ig
```
Check your internet connection
```
curl -sL yabs.sh | bash -s -fg
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
git clone https://github.com/Stride-Labs/stride.git && cd stride
git checkout c53f6c562d9d3e098aab5c27303f41ee055572cb
sh ./scripts-local/build.sh -s $HOME/go/bin
```
Check version
```
strided version --long | head
```
Initialize a node to create the necessary configuration files
```
strided init <name_moniker> --chain-id STRIDE-1
```
Download `genesis file`
```
wget -O $HOME/.stride/config/genesis.json "https://raw.githubusercontent.com/Stride-Labs/testnet/main/poolparty/genesis.json"
```
Check version of a `genesis file`
```
sha256sum ~/.stride/config/genesis.json
```
Check the state of the validator blocks at the initial stage
```
cd && cat .stride/data/priv_validator_state.json
```
`{
  "height": "0",
  "round": 0,
  "step": 0
}`

If the values above are not equal to zero, then execute the command, but if they are equal to zero, then skip this step
```
strided tendermint unsafe-reset-all --home $HOME/.stride
```
Download the `addrbook file`
```
wget -O $HOME/.stride/config/addrbook.json "https://raw.githubusercontent.com/doxe1/testnet-manuals-doxe/Stride/addrbook.json"
```
## Setting up the node configuration
Edit the config so that we no longer use the `chain-id` flag for each CLI command in client.toml
```
strided config chain-id STRIDE-1
```
If necessary, configure the keyring-backend in `client.toml`
```
strided config keyring-backend os
```
Setting the minimum price for gas in `app.toml`
```
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0025ustrd\"/;" ~/.stride/config/app.toml
```
Add seeds/bpeers/peers to `config.toml`
```
external_address=$(wget -qO- eth0.me)
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.stride/config/config.toml
```
```
peers="b11187784240586475422b132a3dcbc970a996dd@stride-node1.poolparty.stridenet.co:26656,2312417b613c44164bf167cb232795e7d8815be7@65.108.76.44:11523,84ff28824a911409e2c24f2f5ede87ae1b859b5f@5.189.178.222:46656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.stride/config/config.toml
```
```
seeds="baee9ccc2496c2e3bebd54d369c3b788f9473be9@seedv1.poolparty.stridenet.co:26656"
```
Increase the number of incoming and outgoing peers for the connection, except for persistent peers in `config.toml`
```
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 100/g' $HOME/.stride/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 100/g' $HOME/.stride/config/config.toml
```
**(Optional)** Configure pruning with one command `app.toml`
```
pruning="custom" && \
pruning_keep_recent="100" && \
pruning_keep_every="0" && \
pruning_interval="50" && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.stride/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.stride/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.stride/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.stride/config/app.toml
```
**(Optional)** Turn off indexing in `config.toml`
```
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.stride/config/config.toml
```
**(Optional)** On/off snapshots in `app.toml`
```
snapshot_interval=1000 && \
sed -i.bak -e "s/^snapshot-interval *=.*/snapshot-interval = \"$snapshot_interval\"/" ~/.stride/config/app.toml
```
By default snapshots are disabled `snapshot-interval=0`
## Create a service file
```
sudo tee /etc/systemd/system/strided.service > /dev/null <<EOF
[Unit]
Description=strided
After=network-online.target

[Service]
User=$USER
ExecStart=$(which strided) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
```
sudo systemctl daemon-reload && \
sudo systemctl enable strided && \
sudo systemctl restart strided && sudo journalctl -u strided -f -o cat
```
:heavy_exclamation_mark:If after start the node cannot connect to peers for a long time, then look for new peers or ask for addrbook.json in discord and so on:heavy_exclamation_mark:

Stop the node, delete the address book and reset node
```
sudo systemctl stop strided
rm $HOME/.stride/config/addrbook.json
strided tendermint unsafe-reset-all --home $HOME/.stride
```
Restart the node
```
sudo systemctl restart strided && journalctl -u strided -f -o cat
```
Create or restore the wallet and save the memnomics and other information as needed

:heavy_exclamation_mark:Don't forget to save yourself a memo, because you are responsible for your wallet:heavy_exclamation_mark:

Set up a wallet
```
strided keys add <name_wallet> --keyring-backend os
```
Recover a wallet
```
strided keys add <name_wallet> --recover --keyring-backend os
```
Connect ledger wallet
```
strided keys add <name_wallet> --ledger 
```
Create a validator

:heavy_exclamation_mark:Don't forget to save `priv_validator_key.json` 
```
strided tx staking create-validator \
--chain-id STRIDE-1 \
--commission-rate 0.05 \
--commission-max-rate 0.2 \
--commission-max-change-rate 0.1 \
--min-self-delegation "1000000" \
--amount 1000000ustrd \
--pubkey $(strided tendermint show-validator) \
--moniker "<name_moniker>" \
--from <name_wallet> \
--fees 555ustrd
```

