# Canto Mainnet (canto_7700-1)

![image](https://avatars.githubusercontent.com/u/104648009?s=200&v=4)

## <a href="https://canto.io/">🌎 Website </a>
## <a href="https://discord.com/invite/ucRX6XCFbr">💎 Discord </a>
## <a href="https://ping.pub/canto">🚀 Explorer </a>

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
## Install Node

```
git clone https://github.com/Canto-Network/Canto
cd Canto
git checkout v5.0.0
make install
cantod version
# v5.0.0
```
## Update 25.01.23 

```
cd Canto
git pull
git checkout v5.0.0
make install
cantod version
# 5.0.0
```
## Initialize the node
```
cantod config keyring-backend test
cantod init Name --chain-id canto_7700-1
```

## Download Genesis
```
wget -O $HOME/.cantod/config/genesis.json "https://raw.githubusercontent.com/Alexmed911/Nodes-Setup-Manuals/main/Canto/genesis.json"
sha256sum $HOME/.cantod/config/genesis.json
# 5048ba449ae348682fd86840452e88bd0812316279697c04ad288a9059f12f59
```
## Create/recover wallet
```
cantod keys add wallet
cantod keys add wallet --recover
```

## Configure Peers/Gas-prices/Indexing
```
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.001acanto"|g' $HOME/.cantod/config/app.toml
peers="19da65350e754586cd2311a5c90032ec2461f07f@65.21.202.116:26656,b0a85e37973ba1e2a304c9c5e65c454c218eb2c0@canto.p2p.chandrastation.com:26656,16ca056442ffcfe509cee9be37817370599dcee1@147.182.255.149:26656"
sed -i -e 's|^seeds *=.*|seeds = "'$seeds'"|; s|^persistent_peers *=.*|persistent_peers = "'$peers'"|' $HOME/.cantod/config/config.toml
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.cantod/config/config.toml
```
## Download Addrbook
```
wget -O $HOME/.cantod/config/addrbook.json "https://raw.githubusercontent.com/Alexmed911/Nodes-Setup-Manuals/main/Canto/addrbook.json"
```
## Pruning
```
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.cantod/config/config.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.cantod/config/config.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.cantod/config/config.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.cantod/config/config.toml
```
## Create Service
```
sudo tee /etc/systemd/system/cantod.service > /dev/null <<EOF
[Unit]
Description=Cantod
After=network-online.target

[Service]
User=$USER
ExecStart=$(which cantod) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
## Start Node Service
```
sudo systemctl daemon-reload
sudo systemctl enable cantod
sudo systemctl restart cantod && sudo journalctl -u cantod -f -o cat
```
## State-Sync
```
cp $HOME/.cantod/data/priv_validator_state.json $HOME/.cantod/priv_validator_state.json.backup
cantod tendermint unsafe-reset-all --home $HOME/.cantod --keep-addr-book
SNAP_RPC="https://canto-rpc.polkachu.com:443"
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 500)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.cantod/config/config.toml

mv $HOME/.cantod/priv_validator_state.json.backup $HOME/.cantod/data/priv_validator_state.json
sudo systemctl restart cantod && journalctl -u cantod -f -o cat
```
## Create validator
```
cantod tx staking create-validator \
    --from=wallet \
    --chain-id=canto_7700-1 \
    --moniker="Name" \
    --commission-rate="0.07" \
    --commission-max-rate="0.20" \
    --commission-max-change-rate="0.01" \
    --details="" \
    --security-contact="" \
    --website="" \
    --identity="" \
    --pubkey=$(cantod tendermint show-validator) \
    --min-self-delegation="1" \
    --amount=1acanto \
    --gas 300000 \
    --fees=200000000000000000acanto 

  
# if use another port --node "tcp://127.0.0.1:$$657"
  ``` 
##  Delegate stake
```
cantod tx staking delegate $Valoper 1000000acanto --from=wallet --chain-id=canto_7700-1 --gas=auto
```
##  Withdraw reward with commision
```
cantod tx distribution withdraw-rewards $Valoper--from=wallet --commission --chain-id=canto_7700-1 --gas=auto
```
##  Balance
```
cantod q bank balances $(cantod keys show wallet -a)
```
##  Reset
```
cantod tendermint unsafe-reset-all --home $HOME/.cantod --keep-addr-book
