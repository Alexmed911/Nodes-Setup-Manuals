# Defund Testnet (defund-private-4)

![image](https://miro.medium.com/max/750/1*ARSUtObxo8xSh2kReWgv2Q.webp)

## <a href="https://defund.app/">🌎 Website </a>
## <a href="https://discord.gg/bWZqS6xBcK">💎 Discord </a>
## <a href="https://defund.explorers.guru/">🚀 Explorer </a>

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
cd $HOME
rm -rf defund
git clone https://github.com/defund-labs/defund.git
cd defund
git checkout v0.2.3
make install
defundd version         
#v0.2.3
```

## Initialize the node
```
defundd config keyring-backend test
defundd config chain-id defund-private-4
defundd init $Moniker-name --chain-id defund-private-4
```

## Download Genesis
```
cd $HOME/.defund/config
curl -s https://raw.githubusercontent.com/defund-labs/testnet/main/defund-private-4/genesis.json > ~/.defund/config/genesis.json
sha256sum $HOME/.defund/config/genesis.json
# db13a33fbb4048c8701294de79a42a2b5dff599d653c0ee110390783c833208b
```
## Create/recover wallet
```
defundd keys add wallet
defundd keys add wallet --recover
```

## Configure Peers/Gas-prices
```
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.0001ufetf"|g' $HOME/.defund/config/app.toml
peers="f03f3a18bae28f2099648b1c8b1eadf3323cf741@162.55.211.136:26656"
sed -i -e 's|^seeds *=.*|seeds = "'$seeds'"|; s|^persistent_peers *=.*|persistent_peers = "'$peers'"|' $HOME/.defund/config/config.toml
```
## Download Addrbook
```
wget -O $HOME/.defund/config/addrbook.json "https://raw.githubusercontent.com/Alexmed911/Nodes-Setup-Manuals/main/Defund/addrbook.json"
```
## Pruning
```
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.defund/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.defund/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.defund/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.defund/config/app.toml
```
## Create Service
```
sudo tee /etc/systemd/system/defundd.service > /dev/null << EOF
[Unit]
Description=Defund Node
After=network-online.target
[Service]
User=$USER
ExecStart=$(which defundd) start
Restart=on-failure
RestartSec=5
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
```
## Start Node Service
```
sudo systemctl daemon-reload
sudo systemctl enable defundd
sudo systemctl restart defundd && sudo journalctl -u defundd -f -o cat
```
## Snapshot
```
cd $HOME/.defund
sudo systemctl stop defundd
cp $HOME/.defund/data/priv_validator_state.json $HOME/.defund/priv_validator_state.json.backup
defundd tendermint unsafe-reset-all --home $HOME/.defund --keep-addr-book
SNAP_NAME=$(curl -s http://snapshots.autostake.net/defund-private-4/ | egrep -o ">defund-private-4.*.tar.lz4" | tr -d ">" | tail -1)
wget -O - http://snapshots.autostake.net/defund-private-4/$SNAP_NAME | lz4 -d | tar -xvf -
mv $HOME/.defund/priv_validator_state.json.backup $HOME/.defund/data/priv_validator_state.json
sudo systemctl restart defundd && journalctl -u defundd -f -o cat
defundd status 2>&1 | jq .SyncInfo
```
## Create validator
```
defundd tx staking create-validator  \      
  --amount 1000000ufetf \
  --from wallet \
  --commission-max-change-rate "0.1" \
  --commission-max-rate "0.2" \
  --commission-rate "0.05" \
  --min-self-delegation "1" \
  --pubkey  $(defundd tendermint show-validator) \
  --moniker=Moniker \
  --chain-id defund-private-4 \
  --website="" \
  --identity="" \
  --details="" \
  -y
  
   # if use another port --node "tcp://127.0.0.1:$$657"
```
##  Delegate stake
```
defundd tx staking delegate $Valoper 10000000ufetf --from=wallet --chain-id=defund-private-4 --gas=auto
```
##  Balance
```
defundd q bank balances $(defundd keys show wallet -a)
```
##  Reset
```
defundd tendermint unsafe-reset-all --home $HOME/.defund --keep-addr-book
