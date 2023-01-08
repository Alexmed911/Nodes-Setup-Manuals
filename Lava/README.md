# Lava Network Testnet (lava-testnet-1)

![image](https://mirror-media.imgix.net/publication-images/gq4KFLkILtGC7LZhNp9Gr.jpeg?height=800&width=1600&h=800&w=1600&auto=compress)

## <a href="https://lavanet.xyz/">🌎 Website </a>
## <a href="https://discord.gg/mCBzfEbYcF">💎 Discord </a>
## <a href="https://lava.explorers.guru/">🚀 Explorer </a>

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
git clone https://github.com/K433QLtr6RA9ExEq/GHFkqmTzpdNLDd6T.git
wget https://lava-binary-upgrades.s3.amazonaws.com/testnet/v0.4.0/lavad
chmod +x lavad
mv lavad $HOME/go/bin/
lavad version         
#0.4.0-rc2-e2c69db
```
## Initialize the node
```
lavad init MONIKER --chain-id lava-testnet-1
```

## Download Genesis
```
wget -O $HOME/.lava/config/genesis.json "https://raw.githubusercontent.com/Alexmed911/Nodes-Setup-Manuals/main/Lava/genesis.json"
sha256sum $HOME/.lava/config/genesis.json
# 72170a8a7314cb79bc57a60c1b920e26457769667ce5c2ff0595b342c0080d78
```
## Create/recover wallet
```
lavad keys add [key_name]
lavad keys add [key_name] --recover
```

## Configure Peers/Gas-prices/Indexing
```
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0ulava"|g' $HOME/.lava/config/app.toml
peers="ac7cefeff026e1c616035a49f3b00c78da63c2e9@18.215.128.248:26656,6c988ad39fef48abd5504fda547d561fb8a60c3a@130.185.119.243:33656,2c2353c872b0c5af562c518b1aa48a2649a4c927@65.108.199.62:11656,4f9120f706512162fbe4f39aac78b9924efbec58@65.109.92.235:11036,f9190a58670c07f8202abfd9b5b14187b18d755b@144.76.97.251:27656,f120685de6785d8ee0eadfca42407c6e10593e74@144.76.90.130:32656,6641a193a7004447c1b49b8ffb37a90682ce0fb9@65.108.78.116:13656,c19965fe8a1ea3391d61d09cf589bca0781d29fd@162.19.217.52:26656,0516c4d11552b334a683bdb4410fa22ef7e3f8ba@65.21.239.60:11656,dabe2e77bd6b9278f484b34956750e9470527ef7@178.18.246.118:26656,24a2bb2d06343b0f74ed0a6dc1d409ce0d996451@188.40.98.169:27656,b7c3cedc778d93296f179373c3bc6a521e4b682e@65.109.69.160:30656,c678ae0fd7b754615e55bba2589a86e60fc8d45c@136.243.88.91:7140,a65de5f01394199366c182a18d718c9e3ef7f981@159.148.146.132:26656,5c2a752c9b1952dbed075c56c600c3a79b58c395@lava.testnet.peer.autostake.net:27066"
sed -i -e 's|^seeds *=.*|seeds = "'$seeds'"|; s|^persistent_peers *=.*|persistent_peers = "'$peers'"|' $HOME/.lava/config/config.toml
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.lava/config/config.toml
```
## Download Addrbook
```
wget -O $HOME/.lava/config/addrbook.json "https://raw.githubusercontent.com/Alexmed911/Nodes-Setup-Manuals/main/Lava/addrbook.json"
```
## Pruning
```
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.lava/config/config.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.lava/config/config.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.lava/config/config.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.lava/config/config.toml
```
## Create Service
```
sudo tee /etc/systemd/system/lavad.service > /dev/null <<EOF
[Unit]
Description=Lava
After=network-online.target

[Service]
User=$USER
ExecStart=$(which lavad) start
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
sudo systemctl enable lavad
sudo systemctl restart lavad && sudo journalctl -u lavad -f -o cat
```
## Snapshot
```
cd $HOME/.lava
sudo systemctl stop lavad
cp $HOME/.lava/data/priv_validator_state.json $HOME/.lava/priv_validator_state.json.backup
lavad tendermint unsafe-reset-all --home $HOME/.lava --keep-addr-book
SNAP_NAME=$(curl -s http://snapshots.autostake.net/lava-testnet-1/ | egrep -o ">lava-testnet-1.*.tar.lz4" | tr -d ">" | tail -1)
wget -O - http://snapshots.autostake.net/lava-testnet-1/$SNAP_NAME | lz4 -d | tar -xvf -
mv $HOME/.lava/priv_validator_state.json.backup $HOME/.lava/data/priv_validator_state.json
sudo systemctl restart lavad && journalctl -u lavad -f -o cat
lavad status 2>&1 | jq .SyncInfo
```
## Faucet
```
https://discord.com/channels/963778337904427018/1059851367717556314
```
## Create validator
```
lavad tx staking create-validator \
--from wallet \
--amount 1000000ulava \
--pubkey "$(lavad tendermint show-validator)" \
--chain-id lava-testnet-1 \
--moniker="Name" \
--commission-max-change-rate=0.01 \
--commission-max-rate=1.0 \
--commission-rate=0.07 \
--min-self-delegation="1" \
--website="" \
--identity="" \
--details "" \
--security-contact="" \
--fees=5000ulava 
-y

  
# if use another port --node "tcp://127.0.0.1:$$657"
  ``` 
##  Delegate stake
```
lavad tx staking delegate $Valoper 10000000ulava --from=wallet --chain-id=lava-testnet-1 --gas=auto
```
##  Balance
```
lavad q bank balances $(lavad keys show wallet -a)
```
##  Reset
```
lavad tendermint unsafe-reset-all --home $HOME/.lava --keep-addr-book
