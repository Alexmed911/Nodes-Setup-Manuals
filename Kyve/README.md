# Kyve Testnet (kaon-1)

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
wget https://kyve-korellia.s3.eu-central-1.amazonaws.com/v0.8.0/kyved_linux_amd64.tar.gz
tar -xvzf kyved_linux_amd64.tar.gz
chmod +x chaind
sudo mv chaind $HOME/go/bin/chaind
rm kyved_linux_amd64.tar.gz

```
## Initialize the node
```
kyved init Name --chain-id kaon-1
```

## Download Genesis
```
wget -O $HOME/.kyve/config/genesis.json "https://raw.githubusercontent.com/Alexmed911/Nodes-Setup-Manuals/main/Kyve/genesis.json"
sha256sum $HOME/.kyve/config/genesis.json
# 3532166eb1605057f633ff577b4fc3e57a6dddc46498c5bc6f2f4e8ab0c756b8
```
## Create/recover wallet
```
kyved keys add wallet
kyved keys add wallet --recover
```

## Configure Peers/Gas-prices/Indexing
```
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.001tkyve"|g' $HOME/.kyve/config/app.toml
peers="81.30.157.35:14656,664e06d2d6110c5ba93f8ecfee66f150bad981bf@kyve-testnet-peer.itrocket.net:28656,5f54a853e7224ad32cbe4e5cddead24b512b629f@51.159.191.220:28656,5d79eb04b94300f5a7982e065a6340ba4ebd4da3@45.33.28.253:26656,157e0aca4aa382d62e24ffc7f936a5e8bbf4e90e@207.180.245.116:46656,b2b4479a6cb001ffe39d4a95f31bb6993ae0a256@194.163.190.31:26656,c0c8ed45a6c266c4ebe028788456cb14b44164bb@65.109.37.21:27656,f5a6484b239fdbe3f9c9bad889d737e8a9f153c6@149.102.140.248:46656,cf69d30beecfdd44d497fb56eb61b12bbffaf38f@167.86.72.171:26656,39392cf41c1d7ae8f98b6efaa740dc4abe3002ff@65.109.92.241:20656,72df41dabaa13d194e2aa633b1f9af60c9cbd5a2@45.158.38.38:26656,0a7504c77cbeb0c3ead588972780f4c670f5a377@65.109.135.149:26656"
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
SNAP_RPC=""
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
 --amount 10000000tkyve \
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
 --fees=1009405tkyve \
 --gas=auto \
 --chain-id kaon-1

  
# if use another port --node "tcp://127.0.0.1:$$657"
  ``` 
##  Delegate stake
```
kyved tx staking delegate $Valoper 1000000tkyve --from=wallet --chain-id=kaon-1 --gas=auto
```
##  Withdraw reward with commision
```
kyved tx distribution withdraw-rewards $Valoper--from=wallet --commission --chain-id=kaon-1 --gas=auto
```
##  Balance
```
kyved q bank balances $(kyved keys show wallet -a)
```
##  Reset
```
kyved tendermint unsafe-reset-all --home $HOME/.kyve --keep-addr-book
