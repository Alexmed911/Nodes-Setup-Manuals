# OKP4 Testnet (okp4-nemeton-1)

![image](https://github.com/okp4/.github/raw/main/profile/static/okp4-banner.webp)

## <a href="https://okp4.network/">🌎 Website </a>
## <a href="https://discord.gg/wNEuAx93E7">💎 Discord </a>
## <a href="https://okp4.explorers.guru/">🚀 Explorer </a>

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
git clone https://github.com/okp4/okp4d.git
cd okp4d
git checkout v3.0.0
make install
okp4d version         
#v3.0.0
```
## Initialize the node
```
okp4d config keyring-backend test
okp4d init Name --chain-id okp4-nemeton-1
okp4d config chain-id okp4-nemeton-1
```

## Download Genesis
```
wget -O $HOME/.okp4d/config/genesis.json "https://raw.githubusercontent.com/Alexmed911/Nodes-Setup-Manuals/main/OKP4/genesis.json"
sha256sum $HOME/.okp4d/config/genesis.json
# 2ec25f81cc2abecbc0da3de45b052ea3314d0d658b1b7f4c7b6a48d09254c742
```
## Create/recover wallet
```
okp4d keys add [key_name]
okp4d keys add [key_name] --recover
```

## Configure Peers/Gas-prices/Indexing
```
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.001uknow"|g' $HOME/.okp4d/config/app.toml
peers="9c462b1c0ba63115bd70c3bd4f2935fcb93721d0@65.21.170.3:42656,a4a96019d2fbc1b5df07940cd971585311166acd@65.108.206.118:61356,ee4c5d9a8ac7401f996ef9c4d79b8abda9505400@144.76.97.251:12656,2e85c1d08cfca6982c74ef2b67251aa459dd9b2f@65.109.85.170:43656,264256d32511c512a0a9d4098310a057c9999fd1@okp4.sergo.dev:12233,4ea26ce893d8f4f89a7b49b9bd77e0fbd914e029@65.109.88.162:36656,8d8fdad759361a57121903632adbd66ad072b1ab@okp4-testnet.nodejumper.io:29656,e3c602b146121c88d350bd7e0f6ce8977e1aacff@161.97.122.216:26656,3c805c2dead7b7a3a1d3ba2399d4d62153322413@65.108.2.41:36656,9d1482bc31fb4578a5c7f7f65c4e0aaf2dfc2336@213.239.215.77:34656,a7f1dcf7441761b0e0e1f8c6fdc79d3904c22c01@[2a02:c206:2093:4875::1]:36656,a7f1dcf7441761b0e0e1f8c6fdc79d3904c22c01@38.242.150.63:36656,99f6675049e22a0216af0e2447e7a4c5021874cd@142.132.132.200:28656,9392c27a9a561c31e7a920dc6f577d663c473ef8@154.12.225.88:26656"
sed -i -e 's|^seeds *=.*|seeds = "'$seeds'"|; s|^persistent_peers *=.*|persistent_peers = "'$peers'"|' $HOME/.okp4d/config/config.toml
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.okp4d/config/config.toml
```
## Download Addrbook
```
wget -O $HOME/.okp4d/config/addrbook.json "https://raw.githubusercontent.com/Alexmed911/Nodes-Setup-Manuals/main/OKP4/addrbook.json"
```
## Pruning
```
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.okp4d/config/config.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.okp4d/config/config.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.okp4d/config/config.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.okp4d/config/config.toml
```
## Create Service
```
sudo tee /etc/systemd/system/okp4d.service > /dev/null <<EOF
[Unit]
Description=OKP4 Node
After=network-online.target

[Service]
User=$USER
ExecStart=$(which okp4d) start
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
sudo systemctl enable okp4d
sudo systemctl restart okp4d && sudo journalctl -u okp4d -f -o cat
```
## Snapshot
```
cd $HOME
sudo systemctl stop okp4d
cp $HOME/.okp4d/data/priv_validator_state.json $HOME/.okp4d/priv_validator_state.json.backup
okp4d tendermint unsafe-reset-all --home $HOME/.okp4d --keep-addr-book
curl -L https://snap.nodeist.net/t/okp4/okp4.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME/.okp4d --strip-components 2
mv $HOME/.okp4d/priv_validator_state.json.backup $HOME/.okp4d/data/priv_validator_state.json
sudo systemctl restart okp4d && journalctl -u okp4d -f -o cat
okp4d status 2>&1 | jq .SyncInfo
```
## Create validator
```
okp4d tx staking create-validator \
--from wallet \
--amount 1000000uknow \
--pubkey "$(okp4d tendermint show-validator)" \
--chain-id okp4-nemeton-1 \
--moniker="Name" \
--commission-max-change-rate=0.01 \
--commission-max-rate=1.0 \
--commission-rate=0.05 \
--min-self-delegation="1" \
--website="" \
--identity="" \
--details "" \
--security-contact="" \
--fees=1000000uknow
  
   # if use another port --node "tcp://127.0.0.1:$$657"
  ``` 
##  Delegate stake
```
okp4d tx staking delegate $Valoper 10000000uknow --from=wallet --chain-id=okp4-nemeton-1 --gas=auto
```
##  Balance
```
okp4d q bank balances $(okp4d keys show wallet -a)
```
##  Reset
```
okp4d tendermint unsafe-reset-all --home $HOME/.okp4d --keep-addr-book
