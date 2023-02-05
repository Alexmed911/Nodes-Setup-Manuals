# C4E Mainnet (perun-1)

![image](https://c4e.io/wp-content/uploads/2023/01/c4e-video-1357x764.png)

## <a href="https://c4e.io/">🌎 Website </a>
## <a href="https://discord.gg/chain4energy">💎 Discord </a>
## <a href="https://ping.pub/chain4energy">🚀 Explorer </a>

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
## Install Node (upd 03.02.23)

```
git clone --depth 1 --branch  v1.1.0  https://github.com/chain4energy/c4e-chain.git
cd c4e-chain
make install
c4ed version
# v1.1.0
```
## Initialize the node
```
c4ed config keyring-backend test
c4ed init Name --chain-id perun-1
```

## Download Genesis
```
wget -O $HOME/.c4e-chain/config/genesis.json "https://raw.githubusercontent.com/Alexmed911/Nodes-Setup-Manuals/main/C4E/genesis.json"
sha256sum $HOME/.c4e-chain/config/genesis.json
# 6c736993a681a6759d3ec41550995fe04f48dd332d03375d879f3b464c6ceabf
```
## Create/recover wallet
```
c4ed keys add wallet
c4ed keys add wallet --recover
```

## Configure Peers/Gas-prices/Indexing
```
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.001uc4e"|g' $HOME/.c4e-chain/config/app.toml
peers="3c6553a3c45477c2a9902e54069bee7109318b9d@163.172.18.144:26656,68a611fc1d17612e4de6b1232d04568ea3c20a19@77.55.216.80:26656,5b62ff6035d9c8143c0ebf4fe05fa0b22d96bb05@rpc.c4e.ppnv.space:13656,f5d50df79f2aa5a9d18576147f59b8807347b6f9@66.70.178.78:26656,85acd1e5580c950f5ede07c3da4bd814d42cf323@95.179.190.59:26656,fe9a629d1bb3e1e958b2013b6747e3dbbd7ba8d3@149.102.130.176:26656"
sed -i -e 's|^seeds *=.*|seeds = "'$seeds'"|; s|^persistent_peers *=.*|persistent_peers = "'$peers'"|' $HOME/.c4e-chain/config/config.toml
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.c4e-chain/config/config.toml
```
## Download Addrbook
```
wget -O $HOME/.c4e-chain/config/addrbook.json "https://raw.githubusercontent.com/Alexmed911/Nodes-Setup-Manuals/main/C4E/addrbook.json"
```
## Pruning
```
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.c4e-chain/config/config.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.c4e-chain/config/config.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.c4e-chain/config/config.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.c4e-chain/config/config.toml
```
## Create Service
```
sudo tee /etc/systemd/system/c4ed.service > /dev/null <<EOF
[Unit]
Description=C4E
After=network-online.target

[Service]
User=$USER
ExecStart=$(which c4ed) start
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
sudo systemctl enable c4ed
sudo systemctl restart c4ed && sudo journalctl -u c4ed -f -o cat
```
## State-Sync
```
cp $HOME/.c4e-chain/data/priv_validator_state.json $HOME/.c4e-chain/priv_validator_state.json.backup
c4ed tendermint unsafe-reset-all --home $HOME/.c4e-chain --keep-addr-book
SNAP_RPC="https://rpc1.c4e.io:443"
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 500)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.c4e-chain/config/config.toml

mv $HOME/.c4e-chain/priv_validator_state.json.backup $HOME/.c4e-chain/data/priv_validator_state.json
sudo systemctl restart c4ed && journalctl -u c4ed -f -o cat
```
## Create validator
```
c4ed tx staking create-validator \
  --amount 1999000uc4e \
  --from wallet \
  --commission-max-change-rate "0.01" \
  --commission-max-rate "0.2" \
  --commission-rate "0.06" \
  --min-self-delegation "1" \
  --pubkey  $(c4ed tendermint show-validator) \
  --moniker Name \
  --website="" \
  --identity="" \
  --details "" \
  --chain-id perun-1 \
  --fees 250uc4e

  
# if use another port --node "tcp://127.0.0.1:$$657"
  ``` 
##  Delegate stake
```
c4ed tx staking delegate $Valoper 1000000uc4e --from=wallet --fees=250uc4e --gas=auto --chain-id=perun-1 
```
##  Withdraw reward with commision
```
c4ed tx distribution withdraw-rewards $Valoper --from=wallet --commission --fees=250uc4e --chain-id=perun-1
```
##  Balance
```
c4ed q bank balances $(c4ed keys show wallet -a)
```
##  Reset
```
c4ed tendermint unsafe-reset-all --home $HOME/.c4e-chain --keep-addr-book
