# Persistence Testnet (test-core-1)

![image](https://icodrops.com/wp-content/uploads/2020/09/Persistence_cover.jpeg)

## <a href="https://persistence.one/">🌎 Website </a>
## <a href="https://discord.gg/qXRmTTGcYD">💎 Discord </a>
## <a href="https://testnet.ping.pub/test-core-1">🚀 Explorer </a>

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
git clone https://github.com/persistenceOne/persistenceCore.git
cd persistenceCore
git fetch --tags
git checkout v6.0.0-rc4
make install
persistenceCore version         
#v6.0.0-rc4
```
## Update 23.12.22
```
cd $HOME/persistenceCore
git pull
git checkout v6.0.0-rc4
make build
cp build/persistenceCore /root/go/bin
```
## Initialize the node
```
persistenceCore config keyring-backend test
persistenceCore init Name --chain-id test-core-1
```

## Download Genesis
```
wget -O $HOME/.persistenceCore/config/genesis.json "https://raw.githubusercontent.com/Alexmed911/Nodes-Setup-Manuals/main/Persistence/test-core-1/genesis.json"
sha256sum $HOME/.persistenceCore/config/genesis.json
# 66d0ffe2912176907e62e94bbec3b20b8ac4171723378e6054123f513482d782
```
## Create/recover wallet
```
persistenceCore keys add [key_name]
persistenceCore keys add [key_name] --recover
```

## Configure Peers/Gas-prices/Indexing
```
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.0001uxprt"|g' $HOME/.persistenceCore/config/app.toml
peers="f85b3dcefb23eae3109cbc89014123ad6e553676@persistence-testnet-statesync.allthatnode.com:26656,1ea969844bd7c7b4268a450222f278bbc9ca32d0@65.1.133.119:26656,a530147d623ef4cbb9d61d06c5e8ddd04180d972@13.208.223.192:26656, 10e69554d68b3c737a7c6bb55938f38e6b547ea7@220.76.21.184:43006, 642cba81f229c50457008410ab5a7a3e6b7b39fe@85.214.61.70:26656, e6f73a89cce68ca961517cf861dbf294a03ad340@18.179.50.45:26656, d4738dfdeede1047076de9ddcc3bef269bdfb898@35.223.239.9:26656, d1fe16cbd078a56465ad2f02bfbf3c8a22253790@13.125.252.209:26656, 723218672704e92a65100ddc28cd5719ada07686@3.25.196.255:26656, e46e42065d8fb108b8f2add2539b16b01f0544f4@13.244.233.149:26656,1ea969844bd7c7b4268a450222f278bbc9ca32d0@65.1.133.119:26656"
sed -i -e 's|^seeds *=.*|seeds = "'$seeds'"|; s|^persistent_peers *=.*|persistent_peers = "'$peers'"|' $HOME/.persistenceCore/config/config.toml
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.persistenceCore/config/config.toml
```
## Download Addrbook
```
wget -O $HOME/.persistenceCore/config/addrbook.json "https://raw.githubusercontent.com/Alexmed911/Nodes-Setup-Manuals/main/Persistence/test-core-1/addrbook.json"
```
## Pruning
```
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.persistenceCore/config/config.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.persistenceCore/config/config.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.persistenceCore/config/config.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.persistenceCore/config/config.toml
```
## Create Service
```
sudo tee /etc/systemd/system/persistenceCore.service > /dev/null <<EOF
[Unit]
Description=persistenceCore
After=network-online.target

[Service]
User=$USER
ExecStart=$(which persistenceCore) start
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
sudo systemctl enable persistenceCore
sudo systemctl restart persistenceCore && sudo journalctl -u persistenceCore -f -o cat
```
## Faucet
```
https://www.allthatnode.com/faucet/persistence.dsrv

```
## Snapshot
```
cd $HOME
sudo systemctl stop persistenceCore
cp $HOME/.persistenceCore/data/priv_validator_state.json $HOME/.persistenceCore/priv_validator_state.json.backup
rm -rf $HOME/.persistenceCore/data
SNAP_NAME=$(curl -s http://snapshots.autostake.net/test-core-1/ | egrep -o ">test-core-1.*.tar.lz4" | tr -d ">" | tail -1)
wget -O - http://snapshots.autostake.net/test-core-1/$SNAP_NAME | lz4 -dc - | tar -xf - -C  $HOME/.persistenceCore
mv $HOME/.persistenceCore/priv_validator_state.json.backup $HOME/.persistenceCore/data/priv_validator_state.json
sudo systemctl restart persistenceCore && journalctl -u persistenceCore -f -o cat
persistenceCore status 2>&1 | jq .SyncInfo
```
## Create validator
```
persistenceCore tx staking create-validator \
--from wallet \
--amount 999000uxprt \
--pubkey "$(persistenceCore tendermint show-validator)" \
--chain-id test-core-1 \
--moniker="Name" \
--commission-max-change-rate=0.01 \
--commission-max-rate=1.0 \
--commission-rate=0.05 \
--min-self-delegation="1" \
--website="" \
--identity="" \
--details "" \
--security-contact="" \
--fees=30uxprt
  
   # if use another port --node "tcp://127.0.0.1:$$657"
