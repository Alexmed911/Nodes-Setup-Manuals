# BlockX Mainnet (blockx_100-1)

![image](https://253241264-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXnmMY63g38TZM2uGNbd1%2Fuploads%2F2cI5N6Cy3soSzgdbYi8F%2Ft.png?alt=media&token=0faf5a35-fcff-4480-9d98-1cdf5baca449)

## <a href="https://www.blockxnet.com/">🌎 Website </a>
## <a href="https://discord.gg/479QrBHbq2">💎 Discord </a>
## <a href="https://ping.blockxnet.com/blockx">🚀 Explorer </a>

# Manual Setup

## Install dependencies
```
sudo apt update && sudo apt upgrade -y
sudo apt install curl tar unzip wget tmux clang lz4 pkg-config libssl-dev jq build-essential git make ncdu gcc jq chrony liblz4-tool -y
```
## Install Go
```
ver="1.20.5" && \
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
rm "go$ver.linux-amd64.tar.gz" && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile && \
source $HOME/.bash_profile && \
go version    
#1.20.5
```
## Install Node

```
cd $HOME
curl -LO https://github.com/defi-ventures/blockx-node-public-compiled/releases/download/v10.0.0/blockxd
chmod +x blockxd
mkdir -p $HOME/go/bin/
mv blockxd $HOME/go/bin/
```
## Initialize the node
```
nolusd config keyring-backend file
nolusd init Name --chain-id blockx_100-1
```

## Download Genesis
```
wget -O $HOME/.blockxd/config/genesis.json "https://raw.githubusercontent.com/Alexmed911/Nodes-Setup-Manuals/main/BlockX/genesis.json"
sha256sum $HOME/.blockxd/config/genesis.json
# 363ce0d9f813c51e0cca3218049354b7472b9aa1bf0325cbd9e08f2562eb5b86
```
## Create/recover wallet
```
nolusd keys add wallet
nolusd keys add wallet --recover
```

## Configure Peers/Indexing
```
peers="bc152258668e673a3b63f964fa75afdd478078c7@mainnet-blockx.konsortech.xyz:39656,72639ce4ce7e0260d7ae129e6acc07dcb54d6af1@rpc.blockx.indonode.net:20656,ed5384bd984a04f19aeb7e17699c061bffd16c41@88.99.59.227:11632"
sed -i -e 's|^seeds *=.*|seeds = "'$seeds'"|; s|^persistent_peers *=.*|persistent_peers = "'$peers'"|' $HOME/.blockxd/config/config.toml
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.blockxd/config/config.toml
```
## Download Addrbook
```
wget -O $HOME/.blockxd/config/addrbook.json "https://raw.githubusercontent.com/Alexmed911/Nodes-Setup-Manuals/main/Nolus Protocol/addrbook.json"
```
## Pruning
```
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.blockxd/config/config.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.blockxd/config/config.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.blockxd/config/config.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.blockxd/config/config.toml
```
## Create Service
```
tee /etc/systemd/system/blockxd.service > /dev/null <<EOF
[Unit]
Description=BlockX
After=network-online.target

[Service]
User=$USER
ExecStart=$(which blockxd) start
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
sudo systemctl enable blockxd
sudo systemctl restart blockxd && sudo journalctl -u blockxd -f -o cat
```
## Shapshot ~3GB updated every 24 hours
```
apt install lz4
sudo systemctl stop blockxd
cp $HOME/.blockxd/data/priv_validator_state.json $HOME/.blockxd/priv_validator_state.json.backup
rm -rf $HOME/.blockxd/data
curl -L http://65.109.52.56/blockx.tar.lz4 | tar -Ilz4 -xf - -C $HOME/.blockxd
mv $HOME/.blockxd/priv_validator_state.json.backup $HOME/.blockxd/data/priv_validator_state.json
sudo systemctl restart blockxd && journalctl -u nolusd -f -o cat
```
## Create validator
```
blockxd tx staking create-validator \
    --from=wallet \
    --chain-id=blockx_100-1 \
    --moniker="Name" \
    --commission-rate="0.07" \
    --commission-max-rate="0.20" \
    --commission-max-change-rate="0.01" \
    --details="" \
    --security-contact="" \
    --website="" \
    --identity="" \
    --pubkey=$(blockxd tendermint show-validator) \
    --min-self-delegation="1" \
    --amount=1bcx \
    --gas 300000 
 

  
# if use another port --node "tcp://127.0.0.1:$$657"
  ``` 
##  Delegate stake
```
blockxd tx staking delegate $Valoper 1bcx --from=wallet --chain-id=blockx_100-1 --gas=auto
```
##  Withdraw reward with commision
```
blockxd tx distribution withdraw-rewards $Valoper--from=wallet --commission --chain-id=blockx_100-1 --gas=auto
```
##  Balance
```
blockxd q bank balances $(blockxd keys show wallet -a)
```
##  Reset
```
blockxd tendermint unsafe-reset-all --home $HOME/.blockxd --keep-addr-book
