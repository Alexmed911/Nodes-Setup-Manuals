# Persistence Testnet (test-core-1)

![image](https://www.marketbeat.com/logos/cryptocurrencies/persistence-XPRT.png?v=2022-10-05)

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
rm -rf defund
git clone https://github.com/defund-labs/defund.git
cd defund
git checkout v0.2.1
make install
defundd version         
#v0.2.1
```

## Initialize the node
```
defundd config keyring-backend test
defundd config chain-id defund-private-3
defundd init $Moniker-name --chain-id defund-private-3
```

## Download Genesis
```
wget -O defund-private-3-gensis.tar.gz https://github.com/defund-labs/testnet/raw/main/defund-private-3/defund-private-3-gensis.tar.gz
sudo tar -xvzf defund-private-3-gensis.tar.gz -C $HOME/.defund/config
rm defund-private-3-gensis.tar.gz
sha256sum $HOME/.defund/config/genesis.json
# 1a10121467576ab6f633a14f82d98f0c39ab7949102a77ab6478b2b2110109e3
```
## Create/recover wallet
```
defundd keys add wallet
defundd keys add wallet --recover
```

## Configure Peers/Gas-prices
```
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.0001ufetf"|g' $HOME/.defund/config/app.toml
peers="263616dba779061a18ded71dddb92928ea27a4ba@43.154.83.15:26656,e108c39c307864acbeceda3f4b2c77c99ec1bddd@185.16.38.136:36656,e4677ff91a0bfec8949de0c2d531b4bbffcb0ceb@92.119.112.231:36656,85b021ed71173a0825736891b06592a8eee7b4ca@43.156.112.45:26656,bdcaabb2384b1a59d12fbd57dd1d74a58edaf1b2@175.24.183.235:26656,45b50b7ad8df4d2661fc6f510bd9d490b5ec253d@43.134.202.178:26656,43452645f84db6827452f32869ddf3ce585937c5@43.156.111.103:26656,257de7d6825037b6c6de16aac4ebb9efd641b8a6@43.156.111.241:26656,58aef46a0286a6d50a7f687bfc35d62f85feec10@107.174.63.166:26656,c8fb3ab19dfac9f75085cb5e4fff36845773d8a6@43.154.60.157:26656,77b3dcacd513f7f7fa1b0247d716f464ad61e94d@65.109.65.210:34656,966e31c78c08aae8c74aa12702126141fb9cef7a@185.165.240.179:24666,92b164431c37b1b8e8cb66cbabcd688108c7479c@43.130.228.99:26656,38d23d7332b035eae29ba0abda13d32906c78c09@65.108.159.90:26656,ce62e6e53805ceae1f8f1087c5f7f6da13049cec@43.130.242.40:26656,53e2240528947ff8f7b037d347b7258f05ce88f0@89.179.68.98:27656"
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
## Sync
```
defundd status 2>&1 | jq .SyncInfo
defundd status 2>&1 | jq .NodeInfo
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
  --chain-id defund-private-3 \
  --website="" \
  --identity="" \
  --details="" \
  -y
  
   # if use another port --node "tcp://127.0.0.1:$$657"
