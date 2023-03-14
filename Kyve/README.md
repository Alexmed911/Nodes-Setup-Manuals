# Kyve Mainnet (kyve-1)

![image](https://miro.medium.com/max/1400/1*2u3ic7iltVQiH7nmUceZjw.png)

## <a href="https://www.kyve.network/">🌎 Website </a>
## <a href="https://discord.gg/Z5Gcj5yq72">💎 Discord </a>
## <a href="https://kyve.explorers.guru/">🚀 Explorer </a>

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
cd $HOME
git clone https://github.com/KYVENetwork/chain.git && cd chain
git checkout v1.0.0
make install
kyved version
#v1.0.0

```
## Initialize the node
```
kyved init Name --chain-id kyve-1
```

## Download Genesis
```
wget -O $HOME/.kyve/config/genesis.json "https://raw.githubusercontent.com/Alexmed911/Nodes-Setup-Manuals/main/Kyve/genesis.json"
sha256sum $HOME/.kyve/config/genesis.json
# 1dc3ec916f49ef8c221851566aca12a3f914b23afb3ab35067fc8a8d5f59c2ee
```
## Create/recover wallet
```
kyved keys add wallet
kyved keys add wallet --recover
```

## Configure Peers/Gas-prices/Indexing
```
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0ukyve"|g' $HOME/.kyve/config/app.toml
peers="38ec29fb7b8babdb592fbaa7131cdbb72ac72476@5.9.61.78:15656,25da6253fc8740893277630461eb34c2e4daf545@3.76.244.30:26656"
sed -i -e 's|^seeds *=.*|seeds = "'$seeds'"|; s|^persistent_peers *=.*|persistent_peers = "'$peers'"|' $HOME/.kyve/config/config.toml
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.kyve/config/config.toml
```
## Download Addrbook
```
wget -O $HOME/.kyve/config/addrbook.json "https://raw.githubusercontent.com/Alexmed911/Nodes-Setup-Manuals/main/Kyve/addrbook.json"
```
## Pruning
```
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.kyve/config/config.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.kyve/config/config.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.kyve/config/config.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.kyve/config/config.toml
```
## Create Service
```
tee /etc/systemd/system/kyved.service > /dev/null <<EOF
[Unit]
Description=Kyve
After=network-online.target

[Service]
User=$USER
ExecStart=$(which kyved) start
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
sudo systemctl enable kyved
sudo systemctl restart kyved && sudo journalctl -u kyved -f -o cat
```
## State-Sync
```
cp $HOME/.kyve/data/priv_validator_state.json $HOME/.kyve/priv_validator_state.json.backup
kyved tendermint unsafe-reset-all --home $HOME/.kyve --keep-addr-book
SNAP_RPC="SOON"
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 500)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.kyve/config/config.toml

mv $HOME/.nolus/priv_validator_state.json.backup $HOME/.nolus/data/priv_validator_state.json
sudo systemctl restart kyved && journalctl -u kyved -f -o cat
```
## Create validator
```
kyved tx staking create-validator \
 --amount 10000000ukyve \
 --from wallet \
 --commission-max-change-rate "0.1" \
 --commission-max-rate "0.2" \
 --commission-rate "0.1" \
 --min-self-delegation "1" \
 --pubkey $(kyved tendermint show-validator) \
 --moniker Name \
 --details="" \
 --website="" \
 --identity="" \
 --fees=1009405ukyve \
 --gas=auto \
 --chain-id kyve-1

  
# if use another port --node "tcp://127.0.0.1:$$657"
  ``` 
##  Delegate stake
```
kyved tx staking delegate $Valoper 1000000ukyve --from=wallet --chain-id=kaon-1 --gas=auto
```
##  Withdraw reward with commision
```
kyved tx distribution withdraw-rewards $Valoper--from=wallet --commission --chain-id=kyve-1 --gas=auto
```
##  Balance
```
kyved q bank balances $(kyved keys show wallet -a)
```
##  Reset
```
kyved tendermint unsafe-reset-all --home $HOME/.kyve --keep-addr-book
