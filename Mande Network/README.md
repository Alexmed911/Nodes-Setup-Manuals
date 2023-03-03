# Mande Network Testnet (mande-testnet-2)

![image](https://pbs.twimg.com/profile_banners/1572091322374451200/1663658882/1500x500)

## <a href="https://www.mande.network/">🌎 Website </a>
## <a href="https://discord.gg/9Ugch3fRC2">💎 Discord </a>
## <a href="https://explorer.stavr.tech/mande-chain">🚀 Explorer </a>

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
wget "https://raw.githubusercontent.com/Alexmed911/Nodes-Setup-Manuals/main/Mande Network/mande-chaind.tar.gz"
tar -xzvf mande-chaind.tar.gz && chmod +x mande-chaind
mv mande-chaind go/bin/
mande-chaind version
# v1.2.2
```
## Initialize the node
```
mande-chaind init Name --chain-id mande-testnet-2
mande-chaind config chain-id mande-testnet-2
```

## Download Genesis
```
wget -O $HOME/.mande-chain/config/genesis.json "https://raw.githubusercontent.com/Alexmed911/Nodes-Setup-Manuals/main/Mande Network/genesis.json"
sha256sum $HOME/.mande-chain/config/genesis.json
# 7b139aea9b1d4210ab89078bd2ec2cd28b86d10870232d07dabdf5d5472f472d
```
## Create/recover wallet
```
mande-chaind keys add wallet
mande-chaind keys add wallet --recover
```

## Configure Peers/Gas-prices/Indexing
```
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.005mand"|g' $HOME/.mande-chain/config/app.toml
peers="dbd1f5b01f010b9e6ae6d9f293d2743b03482db5@34.171.132.212:26656,1d1da5742bdd281f0829124ec60033f374e9ddac@34.170.16.69:26656"
sed -i -e 's|^seeds *=.*|seeds = "'$seeds'"|; s|^persistent_peers *=.*|persistent_peers = "'$peers'"|' $HOME/.mande-chain/config/config.toml
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.mande-chain/config/config.toml
```
## Download Addrbook
```
wget -O $HOME/.mande-chain/config/addrbook.json "https://raw.githubusercontent.com/Alexmed911/Nodes-Setup-Manuals/main/Mande Network/addrbook.json"
```
## Pruning
```
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.nolus/config/config.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.nolus/config/config.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.nolus/config/config.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.nolus/config/config.toml
```
## Create Service
```
tee /etc/systemd/system/mande-chaind.service > /dev/null <<EOF
[Unit]
Description=Mande
After=network-online.target

[Service]
User=$USER
ExecStart=$(which mande-chaind) start
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
sudo systemctl enable mande-chaind
sudo systemctl restart mande-chaind && sudo journalctl -u mande-chaind -f -o cat
```
## State-Sync
```
cp $HOME/.mande-chain/data/priv_validator_state.json $HOME/.mande-chain/priv_validator_state.json.backup
mande-chaind tendermint unsafe-reset-all --home $HOME/.mande-chain --keep-addr-book
SNAP_RPC=""
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 500)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.nolus/config/config.toml

mv $HOME/.mande-chain/priv_validator_state.json.backup $HOME/.mande-chain/data/priv_validator_state.json
sudo systemctl restart mande-chaind && journalctl -u mande-chaind -f -o cat
```
## Create validator
```
mande-chaind tx staking create-validator \
--chain-id mande-testnet-2 \
--amount 0cred \
--pubkey "$(mande-chaind tendermint show-validator)" \
--details "" \
--website="" \
--identity="" \
--from=wallet \
--moniker=Name \
--fees=1000mand \
--gas-adjustment=1.15

  
# if use another port --node "tcp://127.0.0.1:$$657"
  ``` 
##  Delegate stake
```
mande-chaind tx staking delegate $Valoper 1000000unls --from=wallet --chain-id=mande-testnet-2 --gas=auto
```
##  Withdraw reward with commision
```
mande-chaind tx distribution withdraw-rewards $Valoper--from=wallet --commission --chain-id=mande-testnet-2 --gas=auto
```
##  Balance
```
mande-chaind q bank balances $(mande-chaind keys show wallet -a)
```
##  Reset
```
mande-chaind tendermint unsafe-reset-all --home $HOME/.mande-chain --keep-addr-book
