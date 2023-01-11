# Konstellation Mainnet (darchub)

![image](https://ambcrypto.com/wp-content/uploads/2019/06/1RZEWysSh_gGygPwhQPfokA-e1560867196464.png)

## <a href="https://konstellation.tech/">🌎 Website </a>
## <a href="https://discord.gg/fF528PZd">💎 Discord </a>
## <a href="https://www.mintscan.io/konstellation">🚀 Explorer </a>

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
git clone https://github.com/knstl/konstellation/
cd konstellation
git checkout v0.6.2
make build
cd build
mv knstld home/go/bin/
knstld version
#v0.6.2
```
## Initialize the node
```
knstld init Name --chain-id darchub
```

## Download Genesis
```
wget -O $HOME/.knstld/config/genesis.json "https://raw.githubusercontent.com/Alexmed911/Nodes-Setup-Manuals/main/Konstellation/genesis.json"
sha256sum $HOME/.knstld/config/genesis.json
# f0e98b0749801ba0b2ea5f1eeff72df847ec78bde9b8ff9ca114bedc8afd8763
```
## Create/recover wallet
```
knstld keys add key_name
knstld keys add key_name --recover
```

## Configure Peers/Gas-prices/Indexing
```
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.001udarc"|g' $HOME/.knstld/config/app.toml
peers=""
sed -i -e 's|^seeds *=.*|seeds = "'$seeds'"|; s|^persistent_peers *=.*|persistent_peers = "'$peers'"|' $HOME/.knstld/config/config.toml
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.knstld/config/config.toml
```
## Download Addrbook
```
wget -O $HOME/.knstld/config/addrbook.json "https://raw.githubusercontent.com/Alexmed911/Nodes-Setup-Manuals/main/Konstellation/addrbook.json"
```
## Pruning
```
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.knstld/config/config.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.knstld/config/config.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.knstld/config/config.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.knstld/config/config.toml
```
## Create Service
```
sudo tee /etc/systemd/system/nibid.service > /dev/null <<EOF
[Unit]
Description=Nibiru
After=network-online.target

[Service]
User=$USER
ExecStart=$(which nibid) start
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
sudo systemctl enable knstld
sudo systemctl restart knstld && sudo journalctl -u knstld -f -o cat
```
## Snapshot
```
cd $HOME/.knstld
sudo systemctl stop knstld
cp $HOME/.knstld/data/priv_validator_state.json $HOME/.knstld/priv_validator_state.json.backup
knstld tendermint unsafe-reset-all --home $HOME/.knstld --keep-addr-book
SNAP_NAME=$(curl -s http://snapshots.autostake.net/darchub/ | egrep -o ">darchub.*.tar.lz4" | tr -d ">" | tail -1)
wget -O - http://snapshots.autostake.net/darchub/$SNAP_NAME | lz4 -d | tar -xvf -
mv $HOME/.knstld/priv_validator_state.json.backup $HOME/.knstld/data/priv_validator_state.json
sudo systemctl restart knstld && journalctl -u knstld -f -o cat
knstld status 2>&1 | jq .SyncInfo
```
## Faucet
```
https://discord.com/channels/947911971515293759/984840062871175219
```
## Create validator
```
knstld tx staking create-validator \
--from wallet \
--amount 1000000unibi \
--pubkey "$(knstld tendermint show-validator)" \
--chain-id darchub \
--moniker="Name" \
--commission-max-change-rate=0.01 \
--commission-max-rate=1.0 \
--commission-rate=0.07 \
--min-self-delegation="1" \
--website="" \
--identity="" \
--details "" \
--security-contact="" \
--fees=2000udarc 
-y

  
# if use another port --node "tcp://127.0.0.1:$$657"
  ``` 
##  Delegate stake
```
knstld tx staking delegate $Valoper 1000000udarc --from=wallet --chain-id=darchub --gas=auto
```
##  Withdraw reward with commision
```
knstld tx distribution withdraw-rewards $Valoper--from=wallet --commission --chain-id=darchub --gas=auto

##  Balance
```
knstld q bank balances $(knstld keys show wallet -a)
```
##  Reset
```
knstld tendermint unsafe-reset-all --home $HOME/.knstld --keep-addr-book
