# Realio Network Testnet (realionetwork_1110-2)

![image](https://www.realio.fund/hubfs/Imported_Blog_Media/realio-lmx-cover.jpg)

## <a href="https://www.realio.fund/">🌎 Website </a>
## <a href="https://discord.gg/vKhju3JD">💎 Discord </a>
## <a href="https://testnet-explorer.realio.network/">🚀 Explorer </a>

# Manual Setup

## Install dependencies
```
sudo apt update && sudo apt upgrade -y
sudo apt install curl tar unzip wget tmux clang lz4 pkg-config libssl-dev jq build-essential git make ncdu gcc jq chrony liblz4-tool -y
```
## Install Go
```
ver="1.19.3" && \
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
rm "go$ver.linux-amd64.tar.gz" && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile && \
source $HOME/.bash_profile && \
go version    
#1.19.3
```
## Install Node (Update 26.01.23)

```
git clone https://github.com/realiotech/realio-network.git
cd realio-network
git checkout v0.7.2
make install
realio-networkd version
# v0.7.2
```
## Initialize the node
```
realio-networkd config keyring-backend test
realio-networkd init Name --chain-id realionetwork_1110-2
```

## Download Genesis
```
wget -O $HOME/.realio-network/config/genesis.json "https://raw.githubusercontent.com/Alexmed911/Nodes-Setup-Manuals/main/Realio Network/genesis.json"
sha256sum $HOME/.realio-network/config/genesis.json
# a2f8fae48eb019720ef78524d683a9ca22884dd4e9ba4f8d5b3ac10db1275183
```
## Create/recover wallet
```
realio-networkd keys add wallet
realio-networkd keys add wallet --recover
```

## Configure Peers/Gas-prices/Indexing
```
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.001ario"|g' $HOME/.realio-network/config/app.toml
peers="aa194e9f9add331ee8ba15d2c3d8860c5a50713f@143.110.230.177:26656,13de8696c1a4211beb99896408cb0e9b5c174bac@65.109.34.9:65.109.34.9:36656,aa194e9f9add331ee8ba15d2c3d8860c5a50713f@143.110.230.177:26656"
sed -i -e 's|^seeds *=.*|seeds = "'$seeds'"|; s|^persistent_peers *=.*|persistent_peers = "'$peers'"|' $HOME/.realio-network/config/config.toml
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.realio-network/config/config.toml
```
## Download Addrbook
```
wget -O $HOME/.cantod/config/addrbook.json "https://raw.githubusercontent.com/Alexmed911/Nodes-Setup-Manuals/main/Realio Network/addrbook.json"
```
## Pruning
```
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.realio-network/config/config.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.realio-network/config/config.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.realio-network/config/config.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.realio-network/config/config.toml
```
## Create Service
```
sudo tee /etc/systemd/system/realio-networkd.service > /dev/null <<EOF
[Unit]
Description=Realio
After=network.target

[Service]
User=$USER
Type=simple
ExecStart=$(which realio-networkd) start
Restart=on-failure
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
## Start Node Service
```
sudo systemctl daemon-reload
sudo systemctl enable realio-networkd
sudo systemctl restart realio-networkd && sudo journalctl -u realio-networkd -f -o cat
```
## State-Sync
```
cp $HOME/.realio-network/data/priv_validator_state.json $HOME/.realio-network/priv_validator_state.json.backup
realio-networkd tendermint unsafe-reset-all --home $HOME/.realio-network --keep-addr-book
SNAP_RPC="http://realio.srgts.xyz:37657"
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 500)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)
echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.realio-network/config/config.toml

mv $HOME/.realio-network/priv_validator_state.json.backup $HOME/.realio-network/data/priv_validator_state.json
sudo systemctl restart realio-networkd && journalctl -u realio-networkd -f -o cat
```
## Create validator
```
realio-networkd tx staking create-validator \
  --amount=1000000000000000000ario \
  --pubkey=$(realio-networkd tendermint show-validator) \
  --moniker=Moniker \
  --chain-id=realionetwork_1110-2 \
  --commission-rate="0.10" \
  --commission-max-rate="0.20" \
  --commission-max-change-rate="0.1" \
  --min-self-delegation="1" \
  --fees 5000000000000000ario \
  --website="" \
  --identity="" \
  --details "" \
  --gas 800000 \
  --from=wallet

  
# if use another port --node "tcp://127.0.0.1:$$657"
  ``` 
##  Delegate stake
```
realio-networkd tx staking delegate $Valoper 1000000ario --from=wallet --chain-id=realionetwork_1110-2 --gas=auto
```
##  Withdraw reward with commision
```
realio-networkd tx distribution withdraw-rewards $Valoper--from=wallet --commission --chain-id=realionetwork_1110-2 --gas=auto
```
##  Balance
```
realio-networkd q bank balances $(realio-networkd keys show wallet -a)
```
##  Reset
```
realio-networkd tendermint unsafe-reset-all --home $HOME/.realio-network --keep-addr-book
