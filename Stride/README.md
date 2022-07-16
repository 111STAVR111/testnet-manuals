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
Download Genesis file
```
wget -O $HOME/.stride/config/genesis.json "https://raw.githubusercontent.com/Stride-Labs/testnet/main/poolparty/genesis.json"
```
Check version of a genesis file
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
Download the Addrbook
```
wget -O $HOME/.stride/config/addrbook.json "https://raw.githubusercontent.com/doxe1/testnet-manuals-doxe/Stride/addrbook.json"
```
