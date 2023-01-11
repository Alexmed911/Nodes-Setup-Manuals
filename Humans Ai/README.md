# Humans Testnet (testnet-1)

![image](https://icodrops.com/wp-content/uploads/2021/11/Humans_cover.png)

## <a href="https://humans.ai/">🌎 Website </a>
## <a href="https://discord.gg/humansdotai">💎 Discord </a>
## <a href="https://explorer.humans.zone/humans-testnet">🚀 Explorer </a>

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
git clone https://github.com/humansdotai/humans
cd humans
git checkout v1
go build -o humansd cmd/humansd/main.go
mv humansd /root/go/bin/
humansd version
```
## Initialize the node
```
humansd config keyring-backend test
humansd init Name --chain-id testnet-1
```

## Download Genesis
```
wget -O $HOME/.humans/config/genesis.json "https://raw.githubusercontent.com/Alexmed911/Nodes-Setup-Manuals/main/Humans Ai/genesis.json"
sha256sum $HOME/.humans/config/genesis.json
# f5fef1b574a07965c005b3d7ad013b27db197f57146a12c018338d7e58a4b5cd
```
## Create/recover wallet
```
humansd keys add wallet
humansd keys add wallet --recover
```

## Configure Peers/Gas-prices/Indexing
```
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.001uheart"|g' $HOME/.humans/config/app.toml
peers="2fcbf738e6054862ee14f5db926f9674bd6d081d@135.181.221.186:28656,63b22dc6595a4ad3e84826777b23a371c2bc4d6d@84.54.23.195:26656,96fc064917274a80d43985a5c3440254dcae5dc9@65.108.134.208:36656,2cc7701b7d2a9e0384ad787061edd4e5e63357d3@65.109.34.41:26656,11a72d34dec4f6f0fa2211fafba8c6581247958f@185.250.37.13:26656,a1cdecf36cd910af2535abd25972e8d51ab5b590@91.195.101.77:26656,5f8cd0ff3c46e5faa3a4f8a152ec94823452f9e8@194.163.140.190:26656,3ce506b19fff80a7e0b5f9080c5f8496dd890014@78.159.115.21:26656"
sed -i -e 's|^seeds *=.*|seeds = "'$seeds'"|; s|^persistent_peers *=.*|persistent_peers = "'$peers'"|' $HOME/.humans/config/config.toml
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.humans/config/config.toml
```
## Download Addrbook
```
wget -O $HOME/.humans/config/addrbook.json "https://raw.githubusercontent.com/Alexmed911/Nodes-Setup-Manuals/main/Humans Ai/addrbook.json"
```
## Pruning
```
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.humans/config/config.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.humans/config/config.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.humans/config/config.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.humans/config/config.toml
```
## Create Service
```
sudo tee /etc/systemd/system/humansd.service > /dev/null <<EOF
[Unit]
Description=Humans
After=network-online.target

[Service]
User=$USER
ExecStart=$(which humansd) start
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
sudo systemctl enable humansd
sudo systemctl restart humansd && sudo journalctl -u humansd -f -o cat
```
## Snapshot
```
cd $HOME/.humans
sudo systemctl stop humansd
cp $HOME/.humans/data/priv_validator_state.json $HOME/.humans/priv_validator_state.json.backup
humansd tendermint unsafe-reset-all --home $HOME/.humans --keep-addr-book
SNAP_NAME=$(curl -s https://snapshots.bccnodes.com/testnets/humans/ | egrep -o ">testnet-1.*tar" | tail -n 1 | tr -d '>'); \
wget -O - https://snapshots.bccnodes.com/testnets/humans/${SNAP_NAME} | tar xf -
mv $HOME/.humans/priv_validator_state.json.backup $HOME/.humans/data/priv_validator_state.json
sudo systemctl restart humansd && journalctl -u humansd -f -o cat
humansd status 2>&1 | jq .SyncInfo
```
## Create validator
```
humansd tx staking create-validator \
--from wallet \
--amount 1000000uheart \
--pubkey "$(humansd tendermint show-validator)" \
--chain-id testnet-1 \
--moniker="Name" \
--commission-max-change-rate=0.01 \
--commission-max-rate=1.0 \
--commission-rate=0.07 \
--min-self-delegation="1" \
--website="" \
--identity="" \
--details "" \
--security-contact="" \
--fees=2000uheart 
-y

  
# if use another port --node "tcp://127.0.0.1:$$657"
  ``` 
##  Delegate stake
```
humansd tx staking delegate $Valoper 1000000uheart --from=wallet --chain-id=testnet-1 --gas=auto
```
##  Withdraw reward with commision
```
humansd tx distribution withdraw-rewards $Valoper--from=wallet --commission --chain-id=testnet-1 --gas=auto
```
##  Balance
```
humansd q bank balances $(knstld keys show wallet -a)
```
##  Reset
```
humansd tendermint unsafe-reset-all --home $HOME/.humans --keep-addr-book
