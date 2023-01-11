# Jakal Mainnet (jackal-1)

![image](https://pbs.twimg.com/media/FibOBwjaEAA82gl.jpg)

## <a href="https://jackalprotocol.com/">🌎 Website </a>
## <a href="https://discord.gg/jackal-905823514064453642i">💎 Discord </a>
## <a href="https://ping.pub/jackal">🚀 Explorer </a>

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
git clone https://github.com/JackalLabs/canine-chain 
cd canine-chain
git checkout v1.1.2-hotfix
make install
canined version
# v1.1.2
```
## Initialize the node
```
canined config keyring-backend test
canined init Name --chain-id jackal-1
```

## Download Genesis
```
wget -O $HOME/.canine/config/genesis.json "https://raw.githubusercontent.com/Alexmed911/Nodes-Setup-Manuals/main/Jakal/genesis.json"
sha256sum $HOME/.canine/config/genesis.json
# 851717cefe35004661fea8ff35212f35277f48c88ea0828b1ef6e877e5b4c787
```
## Create/recover wallet
```
canined keys add wallet
canined keys add wallet --recover
```

## Configure Peers/Gas-prices/Indexing
```
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.001ujkl"|g' $HOME/.canine/config/app.toml
peers="2fcbf738e6054862ee14f5db926f9674bd6d081d@135.181.221.186:28656,63b22dc6595a4ad3e84826777b23a371c2bc4d6d@84.54.23.195:26656,96fc064917274a80d43985a5c3440254dcae5dc9@65.108.134.208:36656,2cc7701b7d2a9e0384ad787061edd4e5e63357d3@65.109.34.41:26656,11a72d34dec4f6f0fa2211fafba8c6581247958f@185.250.37.13:26656,a1cdecf36cd910af2535abd25972e8d51ab5b590@91.195.101.77:26656,5f8cd0ff3c46e5faa3a4f8a152ec94823452f9e8@194.163.140.190:26656,3ce506b19fff80a7e0b5f9080c5f8496dd890014@78.159.115.21:26656"
sed -i -e 's|^seeds *=.*|seeds = "'$seeds'"|; s|^persistent_peers *=.*|persistent_peers = "'$peers'"|' $HOME/.canine/config/config.toml
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.canine/config/config.toml
```
## Download Addrbook
```
wget -O $HOME/.canine/config/addrbook.json "https://raw.githubusercontent.com/Alexmed911/Nodes-Setup-Manuals/main/Jakal/addrbook.json"
```
## Pruning
```
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.canine/config/config.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.canine/config/config.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.canine/config/config.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.canine/config/config.toml
```
## Create Service
```
sudo tee /etc/systemd/system/canined.service > /dev/null <<EOF
[Unit]
Description=Canided
After=network-online.target

[Service]
User=$USER
ExecStart=$(which canined) start
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
sudo systemctl enable canined
sudo systemctl restart canined && sudo journalctl -u canined -f -o cat
```
## Snapshot
```
cd $HOME/.canine
sudo systemctl stop canined
cp $HOME/.canine/data/priv_validator_state.json $HOME/.canine/priv_validator_state.json.backup
canined tendermint unsafe-reset-all --home $HOME/.canine --keep-addr-book
SNAP_NAME=$(curl -s http://snapshots.autostake.net/jackal-1/ | egrep -o ">jackal-1.*.tar.lz4" | tr -d ">" | tail -1)
wget -O - http://snapshots.autostake.net/jackal-1/$SNAP_NAME | lz4 -d | tar -xvf -
mv $HOME/.canine/priv_validator_state.json.backup $HOME/.canine/data/priv_validator_state.json
sudo systemctl restart canined && journalctl -u canined -f -o cat
canined status 2>&1 | jq .SyncInfo
```
## Create validator
```
canined tx staking create-validator \
--from wallet \
--amount 1000000ujkl \
--pubkey "$(canined tendermint show-validator)" \
--chain-id jackal-1 \
--moniker="Name" \
--commission-max-change-rate=0.01 \
--commission-max-rate=1.0 \
--commission-rate=0.07 \
--min-self-delegation="1" \
--website="" \
--identity="" \
--details "" \
--security-contact="" \
--fees=2000ujkl 
-y

  
# if use another port --node "tcp://127.0.0.1:$$657"
  ``` 
##  Delegate stake
```
canined tx staking delegate $Valoper 1000000ujkl --from=wallet --chain-id=jackal-1 --gas=auto
```
##  Withdraw reward with commision
```
canined tx distribution withdraw-rewards $Valoper--from=wallet --commission --chain-id=jackal-1 --gas=auto
```
##  Balance
```
canined q bank balances $(canined keys show wallet -a)
```
##  Reset
```
canined tendermint unsafe-reset-all --home $HOME/.canine --keep-addr-book
