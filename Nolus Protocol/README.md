# Nolus Testnet (nolus-rila)

![image](https://preview.redd.it/d369ysgzmi0a1.jpg?width=1200&format=pjpg&auto=webp&v=enabled&s=c95763057eda098eca6b9d6a475d67626472c52f)

## <a href="https://nolus.io/">🌎 Website </a>
## <a href="https://discord.gg/nolus-protocol">💎 Discord </a>
## <a href="https://nolus.explorers.guru/">🚀 Explorer </a>

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
git clone https://github.com/Nolus-Protocol/nolus-core
cd nolus-core
git checkout v0.2.1-testnet
make install
nolusd version
# v0.2.1-testnet
```
## Update 11.03.23

```
cd nolus-core
git pull
git checkout v0.2.1-testnet
make install
nolusd version
# v0.2.1-testnet
```
## Initialize the node
```
nolusd config keyring-backend test
nolusd init Name --chain-id nolus-rila
```

## Download Genesis
```
wget -O $HOME/.nolus/config/genesis.json "https://raw.githubusercontent.com/Nolus-Protocol/nolus-networks/main/testnet/nolus-rila/genesis.json"
sha256sum $HOME/.nolus/config/genesis.json
# d22ea6488afe58478c54afeb2d6b5a45622c797dfd75c91a8653eb1f094173c5
```
## Create/recover wallet
```
nolusd keys add wallet
nolusd keys add wallet --recover
```

## Configure Peers/Gas-prices/Indexing
```
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.001unls"|g' $HOME/.nolus/config/app.toml
peers="a8bb5dfec40d151e5e019059fe11dae1b6597540@185.135.137.80:43656,28cdf59b342cb19fe488e99fab754ccc90c379e3@185.196.21.104:26656,8c5de077ed97fea13f822e0afa9d5720b1ff7e1d@178.63.8.245:26656,246297e8a811fdfa7926ffd6293f314e3d4a8689@84.46.241.73:26656,618030b8fdbf481fce12c4158a6cf24276ec7e02@164.92.246.148:31656,c247c8d544a5dadfd647b78da330a33af20891d6@194.147.58.90:26656,a953cd150743e269bdbec0924604a71e87a89676@68.183.108.239:26656,538e2a3d6e96cd7bc0635eaa3f8f3695f26503a7@65.108.104.167:21656,cb989bd3f416226bfd71631c0348ea38a1df3ec0@65.109.106.91:23656,1bcd62e9094f47577dfdf7be1f96e0726bfaed5a@65.108.140.109:45656"
sed -i -e 's|^seeds *=.*|seeds = "'$seeds'"|; s|^persistent_peers *=.*|persistent_peers = "'$peers'"|' $HOME/.nolus/config/config.toml
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.nolus/config/config.toml
```
## Download Addrbook
```
wget -O $HOME/.nolus/config/addrbook.json "https://raw.githubusercontent.com/Alexmed911/Nodes-Setup-Manuals/main/Nolus Protocol/addrbook.json"
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
tee /etc/systemd/system/nolusd.service > /dev/null <<EOF
[Unit]
Description=Nolus
After=network-online.target

[Service]
User=$USER
ExecStart=$(which nolusd) start
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
sudo systemctl enable nolusd
sudo systemctl restart nolusd && sudo journalctl -u nolusd -f -o cat
```
## Shapshot ~1GB updated every 12 hours
```
apt install lz4
sudo systemctl stop nolusd
cp $HOME/.nolus/data/priv_validator_state.json $HOME/.nolus/priv_validator_state.json.backup
rm -rf $HOME/.nolus/data
curl -L http://65.109.81.119/nolus.tar.lz4 | tar -Ilz4 -xf - -C $HOME/.nolus
mv $HOME/.nolus/priv_validator_state.json.backup $HOME/.nolus/data/priv_validator_state.json
sudo systemctl restart nolusd && journalctl -u nolusd -f -o cat
```
## Create validator
```
nolusd tx staking create-validator \
    --from=wallet \
    --chain-id=nolus-rila \
    --moniker="Name" \
    --commission-rate="0.07" \
    --commission-max-rate="0.20" \
    --commission-max-change-rate="0.01" \
    --details="" \
    --security-contact="" \
    --website="" \
    --identity="" \
    --pubkey=$(nolusd tendermint show-validator) \
    --min-self-delegation="1" \
    --amount=1000000unls \
    --gas 300000 \
    --fees=2000unls 

  
# if use another port --node "tcp://127.0.0.1:$$657"
  ``` 
##  Delegate stake
```
nolusd tx staking delegate $Valoper 1000000unls --from=wallet --chain-id=nolus-rila --gas=auto
```
##  Withdraw reward with commision
```
nolusd tx distribution withdraw-rewards $Valoper--from=wallet --commission --chain-id=nolus-rila --gas=auto
```
##  Balance
```
nolusd q bank balances $(nolusd keys show wallet -a)
```
##  Reset
```
nolusd tendermint unsafe-reset-all --home $HOME/.nolus --keep-addr-book
