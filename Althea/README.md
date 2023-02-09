# Althea Testnet (althea_7357-1)

![image](https://blog.althea.net/content/images/2021/09/newlogo-1-2.png)

## <a href="https://www.althea.net/">🌎 Website </a>
## <a href="https://discord.gg/4FYRYmnW">💎 Discord </a>
## <a href="https://ping.pub/">🚀 Explorer </a>

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
git clone https://github.com/althea-net/althea-chain
cd althea-chain
git checkout v0.3.2
make install
althea version
# v0.3.2
```
## Initialize the node
```
althea config keyring-backend test
althea init Name --chain-id althea_7357-1
```

## Download Genesis
```
wget -O $HOME/.cantod/config/genesis.json "https://raw.githubusercontent.com/Alexmed911/Nodes-Setup-Manuals/main/Althea/genesis.json"
sha256sum $HOME/.cantod/config/genesis.json
# 5048ba449ae348682fd86840452e88bd0812316279697c04ad288a9059f12f59
```
## Create/recover wallet
```
althea keys add wallet
althea keys add wallet --recover
```

## Configure Peers/Gas-prices/Indexing
```
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.00ualthea"|g' $HOME/.althea/config/app.toml
peers="19da65350e754586cd2311a5c90032ec2461f07f@65.21.202.116:26656,b0a85e37973ba1e2a304c9c5e65c454c218eb2c0@canto.p2p.chandrastation.com:26656,16ca056442ffcfe509cee9be37817370599dcee1@147.182.255.149:26656"
sed -i -e 's|^seeds *=.*|seeds = "'$seeds'"|; s|^persistent_peers *=.*|persistent_peers = "'$peers'"|' $HOME/.althea/config/config.toml
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.althea/config/config.toml
```
## Download Addrbook
```
wget -O $HOME/.althea/config/addrbook.json "https://raw.githubusercontent.com/Alexmed911/Nodes-Setup-Manuals/main/Althea/addrbook.json"
```
## Pruning
```
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.althea/config/config.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.althea/config/config.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.althea/config/config.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruninaltheag_interval\"/" $HOME/.althea/config/config.toml
```
## Create Service
```
sudo tee /etc/systemd/system/althea.service > /dev/null <<EOF
[Unit]
Description=althea
After=network-online.target

[Service]
User=$USER
ExecStart=$(which althea) start
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
sudo systemctl enable althea
sudo systemctl restart althea && sudo journalctl -u althea -f -o cat
```
## State-Sync
```
cp $HOME/.althea/data/priv_validator_state.json $HOME/.althea/priv_validator_state.json.backup
althea tendermint unsafe-reset-all --home $HOME/.althea --keep-addr-book
SNAP_RPC=""
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 500)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.althea/config/config.toml

mv $HOME/.cantod/priv_validator_state.json.backup $HOME/.althea/data/priv_validator_state.json
sudo systemctl restart althea && journalctl -u althea -f -o cat
```
## Create validator
```
althea tx staking create-validator \
--amount=1000000ualthea \
--pubkey=$(althea tendermint show-validator) \
--moniker=Name \
--chain-id=althea_7357-1 \
--commission-rate="0.10" \
--commission-max-rate="0.20" \
--commission-max-change-rate="0.1" \
--min-self-delegation="1" \
--fees 500ualthea \
--from=wallet \
--identity="" \
--website="" \
--details="" \
-y 

  
# if use another port --node "tcp://127.0.0.1:$$657"
  ``` 
##  Delegate stake
```
althea tx staking delegate $Valoper 1000000ualthea --from=wallet --fees=250ualthea --gas=250000 --chain-id=althea_7357-1
```
##  Withdraw reward with commision
```
althea tx distribution withdraw-rewards $Valoper --from=wallet --commission --fees=300ualthea --chain-id=althea_7357-1
```
##  Balance
```
althea q bank balances $(cantod keys show wallet -a)
```
##  Reset
```
althea tendermint unsafe-reset-all --home $HOME/.althea --keep-addr-book
