```markdown
# Aligned-Layer-Testnet Setup Guide
```
This guide provides detailed instructions for setting up an Aligned-Layer-Testnet node from scratch on a server.

## Update Your System

First, ensure that your system's package index is updated and that all existing packages are upgraded to their latest versions:


```bash
sudo apt update && sudo apt upgrade -y
sudo apt install make curl git wget htop tmux build-essential jq make lz4 gcc unzip -y
```

## Install Go

Install Go by specifying the version you wish to install. Replace `ver` with the desired version number:

```bash
ver="1.21.5" # replace with the desired version
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=\$PATH:/usr/local/go/bin:\$HOME/go/bin" >> \$HOME/.bash_profile
source \$HOME/.bash_profile
go version
```

## Install Node

Clone the Aligned Layer Tendermint repository, build, and install:

```bash
cd \$HOME
rm -rf \$HOME/aligned_layer_tendermint
git clone --depth 1 --branch v0.0.2 https://github.com/yetanotherco/aligned_layer_tendermint
cd \$HOME/aligned_layer_tendermint/cmd/alignedlayerd
go build
chmod +x alignedlayerd
sudo mv alignedlayerd /usr/local/bin/
cd \$HOME
alignedlayerd version
```

## Initialize Node

Initialize your node with a unique moniker:

```bash
alignedlayerd init NodeName --chain-id alignedlayer
```

## Download Genesis & Addrbook

Fetch the genesis and addrbook files:

```bash
curl -Ls https://snap.nodex.one/alignedlayer-testnet/genesis.json > \$HOME/.alignedlayer/config/genesis.json
curl -Ls https://snap.nodex.one/alignedlayer-testnet/addrbook.json > \$HOME/.alignedlayer/config/addrbook.json
```

## Configure Peers and Gas Prices

Update your node's configuration with the appropriate peers and gas prices. Replace `PEERS` and `STAKETAB_PEER` with actual peer addresses:

```bash
# Configure persistent peers
PEERS="peer_addresses_here"
STAKETAB_PEER="staketab_peer_address_here"
sed -i -e 's|^persistent_peers *=.*|persistent_peers = "'\$STAKETAB_PEER',\$PEERS'"|' \$HOME/.alignedlayer/config/config.toml

# Configure minimum gas prices
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0.0001stake\"|" \$HOME/.alignedlayer/config/app.toml
```

## Create Service

Create a systemd service to manage your node:

```bash
sudo tee /etc/systemd/system/alignedlayerd.service > /dev/null <<EOF
[Unit]
Description=alignedlayerd
After=network-online.target

[Service]
User=root
ExecStart=\$(which alignedlayerd) start
Restart=always
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

## Launch Node

Enable and start the node service:

```bash
sudo systemctl daemon-reload
sudo systemctl enable alignedlayerd
sudo systemctl restart alignedlayerd
sudo journalctl -u alignedlayerd -f -o cat
```

Please replace placeholders like `NodeName`, `ver`, `PEERS`, and `STAKETAB_PEER` with your actual data. Also, check the repository for any updates or changes to the installation process.
```

Ensure to replace placeholders such as `NodeName`, `ver`, `PEERS`, and `STAKETAB_PEER` with your actual values. Also, verify the repository for any updates or modifications in the installation steps.
