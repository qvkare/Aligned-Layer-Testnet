# Aligned-Layer-Testnet

To install the node on a server from scratch, you can follow these steps based on the instructions found in the repository:

Update your system:
sudo apt update && sudo apt upgrade -y

sudo apt install make curl git wget htop tmux build-essential jq make lz4 gcc unzip -y

Install Go: Replace ver with the version number you wish to install.
ver="1.21.5" # replace with the desired version
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
go version

Install Node:
cd $HOME
rm -rf $HOME/aligned_layer_tendermint
git clone --depth 1 --branch v0.0.2 https://github.com/yetanotherco/aligned_layer_tendermint
cd $HOME/aligned_layer_tendermint/cmd/alignedlayerd
go build
chmod +x alignedlayerd
sudo mv alignedlayerd /usr/local/bin/
cd $HOME
alignedlayerd version

Initialize Node: Replace NodeName with your own moniker.
alignedlayerd init NodeName --chain-id alignedlayer

Download Genesis & Addrbook:
curl -Ls https://snap.nodex.one/alignedlayer-testnet/genesis.json > $HOME/.alignedlayer/config/genesis.json
curl -Ls https://snap.nodex.one/alignedlayer-testnet/addrbook.json > $HOME/.alignedlayer/config/addrbook.json

Configure Peers and Gas Prices: Replace PEERS and STAKETAB_PEER with the actual peer addresses.
# Configure persistent peers
PEERS="peer_addresses_here"
STAKETAB_PEER="staketab_peer_address_here"
sed -i -e 's|^persistent_peers *=.*|persistent_peers = "'$STAKETAB_PEER','$PEERS'"|' $HOME/.alignedlayer/config/config.toml

# Configure minimum gas prices
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0.0001stake\"|" $HOME/.alignedlayer/config/app.toml

Create Service:
sudo tee /etc/systemd/system/alignedlayerd.service > /dev/null <<EOF
[Unit]
Description=alignedlayerd
After=network-online.target

[Service]
User=root
ExecStart=$(which alignedlayerd) start
Restart=always
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF

Launch Node:
sudo systemctl daemon-reload
sudo systemctl enable alignedlayerd
sudo systemctl restart alignedlayerd
sudo journalctl -u alignedlayerd -f -o cat

Please ensure you replace placeholders like NodeName, ver, PEERS, and STAKETAB_PEER with your actual data. Also, check the repository for any updates or changes to the installation processes.
