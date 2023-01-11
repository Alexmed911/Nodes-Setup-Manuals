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
peers="a13b5c78c65b785f4189a7873015c47217f2c83c@65.108.13.185:27565,173c43436e2287f3660c344a5fd2386da4a61968@65.109.92.241:11126,f42498ca4d9e62f95115f04ae18fa5ec1c1487f1@65.108.141.109:18656,ebc272824924ea1a27ea3183dd0b9ba713494f83@jackal.mainnet.peer.autostake.net:26906,c2842c76779913e05fa4256e3caab852e1782951@202.61.194.254:60756,4963e1c374624d2c625bdb89821ed0e7290c835b@152.32.185.156:26626,2ec46ff04ebfafc19f505feaaf00943c15bb2757@185.16.38.149:26656,4398bd773ac885b7365de3604eb487be10c54563@185.16.38.210:26906,ff94a29e02de8369faf37c76d3c97684bbd51bd6@185.16.38.165:17556,ecb163fca7436befa3a5694a7d558e89d3f04b2c@65.109.29.150:17656,e6c767e4aafb41c3d67d89ebc0c8257ec837ae50@188.217.162.92:26656,dd3cab79ffae0aed4f519503b66e9403c69eeb14@85.237.193.101:25565,b9cb5ae544ea402ea55529dda039ae4ead83dfcc@213.239.215.77:26656,fe470c3822748792d4b9d1512ba5cc51c5ce4443@185.180.222.181:46656,a77da5b3ce86a5226bae6e7b87964dd4efe8fe46@65.21.170.3:31656,460cf6a14f3fa0f3882400fbdcb80033105cac79@178.154.241.46:26656,7751d16cfa48da0a5bea6f40e9bcc386b4c76c50@51.89.7.184:26638,ef8c470a03f3753df53dad15a435f99d6869f6a7@51.81.107.95:10856,8d59eb5f7ad207e59c06620f6e9e7b6760b56211@65.108.75.107:18656,55df88ae25223565af42ccd6b3b558b8e70bba31@213.239.216.252:26656,4fa82212d657a171b1f4d3f21da33041f5cff9f9@65.21.88.172:31656,bf62b185eef3c185f8ebf81d5cf54bdc064b21d8@159.69.168.245:26656,d0313585956c8e7969993c1577f4969739b19bb7@46.4.88.116:26656,ea76172528bd8a67544766f306f27f351cb199e5@45.76.241.193:26656,d9bfa29e0cf9c4ce0cc9c26d98e5d97228f93b0b@65.109.88.38:37656,6e7d2937b3d29952cf83058b81fad4a1ec3b88e8@195.3.223.204:10756"
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
