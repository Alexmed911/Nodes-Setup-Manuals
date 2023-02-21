# Uptick Network Testnet (uptick_7000-2)

![image](https://miro.medium.com/max/1400/1*aCkSgk39Uhfb-1wzgTy5Pg.png)

## <a href="https://www.uptick.network/">🌎 Website </a>
## <a href="https://discord.gg/479QrBHbq2">💎 Discord </a>
## <a href="https://uptick.explorers.guru/">🚀 Explorer </a>

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
git clone https://github.com/UptickNetwork/uptick.git
cd uptick
git checkout v0.2.5
make install
uptick version         
#0.2.5
```
## Initialize the node
```
uptickd init Name --chain-id uptick_7000-2
```

## Download Genesis
```
wget -O $HOME/.uptickd/config/genesis.json "https://raw.githubusercontent.com/Alexmed911/Nodes-Setup-Manuals/main/Uptick/genesis.json"
sha256sum $HOME/.uptickd/config/genesis.json
# f96764c7ae1bc713b2acc87b5320f2d10ee26716b3daa6cc455cb3a3906f05c2
```
## Create/recover wallet
```
uptickd keys add key_name
uptickd keys add key_name --recover
```

## Configure Peers/Gas-prices/Indexing
```
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.00auptick"|g' $HOME/.uptickd/config/app.toml
peers="1f96655ed716ecace89f06f10bc10fad14b9fe61@51.89.232.234:27916,40ffd59440b11d63bfb8e20cfed5b36f282a06b3@154.12.238.247:31656,507999588745d6021c012b736c795a93348ae0cd@95.214.55.155:20656,38d149fd90fdc0cd3509b697ad65ff9f6f20cd8f@65.108.6.45:60956"
sed -i -e 's|^seeds *=.*|seeds = "'$seeds'"|; s|^persistent_peers *=.*|persistent_peers = "'$peers'"|' $HOME/.uptickd/config/config.toml
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.uptickd/config/config.toml
```
## Download Addrbook
```
wget -O $HOME/.uptickd/config/addrbook.json "https://raw.githubusercontent.com/Alexmed911/Nodes-Setup-Manuals/main/Uptick/addrbook.json"
```
## Pruning
```
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.uptickd/config/config.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.uptickd/config/config.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.uptickd/config/config.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.uptickd/config/config.toml
```
## Create Service
```
sudo tee /etc/systemd/system/uptickd.service > /dev/null <<EOF
[Unit]
Description=Uptick
After=network-online.target

[Service]
User=$USER
ExecStart=$(which uptickd) start
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
sudo systemctl enable uptickd
sudo systemctl restart uptickd && sudo journalctl -u uptickd -f -o cat
```
## Snapshot
```
cd $HOME/.uptickd
sudo systemctl stop uptickd
cp $HOME/.uptickd/data/priv_validator_state.json $HOME/.uptickd/priv_validator_state.json.backup
uptickd tendermint unsafe-reset-all --home $HOME/.uptickd --keep-addr-book
wget $(curl -s https://services.staketab.com/backend/uptick-testnet/ | jq -r .snap_link)
tar -xf $(curl -s https://services.staketab.com/backend/uptick-testnet/ | jq -r .snap_filename) -C $HOME/.uptickd/data/
mv $HOME/.uptickd/priv_validator_state.json.backup $HOME/.uptickd/data/priv_validator_state.json
sudo systemctl restart uptickd && journalctl -u uptickd -f -o cat
uptickd status 2>&1 | jq .SyncInfo
```
## Faucet
```
https://discord.com/channels/781005936260939818/953652276508119060
```
## Create validator
```
uptickd tx staking create-validator \
--from wallet \
--amount 1000000auptick \
--pubkey "$(uptickd tendermint show-validator)" \
--chain-id uptick_7000-2 \
--moniker="Name" \
--commission-max-change-rate=0.01 \
--commission-max-rate=1.0 \
--commission-rate=0.07 \
--min-self-delegation="1" \
--website="" \
--identity="" \
--details "" \
--security-contact="" \
--fees=1000auptick
-y

  
# if use another port --node "tcp://127.0.0.1:$$657"
  ``` 
##  Delegate stake
```
uptickd tx staking delegate $Valoper 10000000auptick --from=wallet --chain-id=uptick_7000-2 --gas=auto
```
##  Balance
```
uptickd q bank balances $(uptickd keys show wallet -a)
```
##  Reset
```
uptickd tendermint unsafe-reset-all --home $HOME/.uptickd --keep-addr-book
