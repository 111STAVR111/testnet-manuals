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

**update the repositories**
```
sudo apt update && sudo apt upgrade -y
```

**install necessary utilities**
```
sudo apt install curl build-essential git wget jq make gcc tmux htop nvme-cli pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip libleveldb-dev -y
```
**check that the hard disks work**
```
curl -sL yabs.sh | bash -s -ig
```
**check your internet**
```
curl -sL yabs.sh | bash -s -fg
```

**Installing Go (One command)**

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


