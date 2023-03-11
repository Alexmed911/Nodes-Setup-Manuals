# Point Mainnet (point_10687-1)

![image](https://assets-global.website-files.com/623b58ac191ec545688aee2e/628ba04fec02e9ca1a719749_social-media.jpg)

## <a href="https://pointnetwork.io/">🌎 Website </a>
## <a href="https://discord.gg/5vaqtg8CzB">💎 Discord </a>
## <a href="https://ping.pub/point">🚀 Explorer </a>

# Manual Setup

## Install dependencies
```
sudo apt update && sudo apt upgrade -y
sudo apt install curl tar unzip wget tmux clang lz4 pkg-config libssl-dev jq build-essential git make ncdu gcc jq chrony liblz4-tool -y
```
## Install Go
```
ver="1.19.4" && \
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
rm "go$ver.linux-amd64.tar.gz" && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile && \
source $HOME/.bash_profile && \
go version    
#1.19.4
```
## Install Node

```
git clone https://github.com/pointnetwork/point-chain
cd point-chain
git checkout tags/v0.0.4
make install
pointd version
# v0.0.4
```
## Initialize the node
```
pointd config keyring-backend test
pointd init Name --chain-id point_10687-1
```

## Download Genesis
```
wget -O $HOME/.pointd/config/genesis.json "https://raw.githubusercontent.com/Alexmed911/Nodes-Setup-Manuals/main/Point/genesis.json"
sha256sum $HOME/.pointd/config/genesis.json
# d655a35633530c3f9d65d0c65ad287b613bba68218fd7c39549b48b958eec14f
```
## Create/recover wallet
```
pointd keys add wallet
pointd keys add wallet --recover
```

## Configure Peers/Gas-prices/Indexing
```
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0apoint"|g' $HOME/.pointd/config/app.toml
peers="e5880e61180a13614d11ae70ef7847598f00cf0a@23.175.146.228:26656,a15ca4998882b74ed8311dd0f473381b4dcd3a79@88.99.161.162:21656,a15ca4998882b74ed8311dd0f473381b4dcd3a79@rpc.point.indonode.net:21656,c2ad2b84e9747c71a269284414b1532db37f8acd@217.182.199.219:26656,bd3dfbaa59111acbeafc35bfed283c7a55f31606@54.39.129.4:26656,fffbb069e5563a0fb2366818d076c62dfac193db@64.226.88.168:26656,8673c1f04c29c464189e8bf29e51fb0b38da2f19@164.92.145.14:26656,a520fffe8b889b9c38a140fd6a5ae5edba264cab@104.248.194.148:26656,acf109a7c590f8121dda230af6feb176d9b31452@65.109.81.119:34656,7d4424db82cbf73d3b8a47b6430b2afaf2873b2b@167.235.229.236:26656,3bcccc00199c8b8e082e55910d3e5e3320134fd6@154.26.136.203:26656,488259415221639f5c082695cffd4c81d299b662@65.108.232.168:14656,f90323d7323b9e709f2a179430b39593cd7a2ab0@5.161.122.253:26656,d8161e37cdc3ca7dbe1379a054f8f6072147ac76@65.108.44.100:28656,a8bd036af7908c5752896441e1612d3073cc9da5@54.64.157.114:26656,5d2dfcc98233973f74280528a2fcba6707035a1d@45.90.92.185:26656,0764cc1fa52a5da1f8f5c0eb92574c737541ff14@145.239.7.44:26656,0742dfb487761f92287028aff129c88b643aa10d@65.21.204.46:55656,57b4e8246e748e6fed2d7585426e9cbdb96fc5cb@65.109.48.11:26656,7d04addcf070dac5c6fa1d3d588ad833dbf7ae30@88.99.100.39:26656,99dbdf9f6736ae822ae0abe43503f9e037f678d9@144.126.139.109:26656,6994b66f2fd1abe76787c1218f9eb18a7ebbe063@185.227.135.88:26656,a98ce5206930c7f78b54005097b261904541af8c@159.223.234.196:26656,4896c8b474560ff359edd9e2a1e705b0513180e2@144.76.97.251:34656,2894ad67ded3715c591949a50a86eca287227f8d@65.109.122.105:61656,cfc8566d7989a08156ce7775c7ba06910a8305f9@161.97.166.6:26656,76af8650397fb49bbe6f68023a8eb9efb61ef7f4@52.14.238.202:26656"
sed -i -e 's|^seeds *=.*|seeds = "'$seeds'"|; s|^persistent_peers *=.*|persistent_peers = "'$peers'"|' $HOME/.pointd/config/config.toml
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.pointd/config/config.toml
```
## Download Addrbook
```
wget -O $HOME/.pointd/config/addrbook.json "https://raw.githubusercontent.com/Alexmed911/Nodes-Setup-Manuals/main/Point/addrbook.json"
```
## Pruning
```
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.pointd/config/config.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.pointd/config/config.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.pointd/config/config.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.pointd/config/config.toml
```
## Create Service
```
tee /etc/systemd/system/pointd.service > /dev/null <<EOF
[Unit]
Description=Point
After=network-online.target

[Service]
User=$USER
ExecStart=$(which pointd) start
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
sudo systemctl enable pointd
sudo systemctl restart pointd && sudo journalctl -u pointd -f -o cat
```
## State-Sync
```
cp $HOME/.pointd/data/priv_validator_state.json $HOME/.pointd/priv_validator_state.json.backup
pointd tendermint unsafe-reset-all --home $HOME/.pointd --keep-addr-book
SNAP_RPC="https://rpc.point.nodestake.top:443"
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 500)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)
echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.pointd/config/config.toml

mv $HOME/.pointd/priv_validator_state.json.backup $HOME/.pointd/data/priv_validator_state.json
sudo systemctl restart pointd && journalctl -u pointd -f -o cat
```
## Create validator
```
pointd tx staking create-validator \
--amount=1000000000000000000apoint \
--pubkey=$(pointd tendermint show-validator) \
--moniker="" \
--chain-id=point_10687-1 \
--commission-rate="0.10" \
--commission-max-rate="0.20" \
--commission-max-change-rate="0.01" \
--min-self-delegation="1" \
--details="" \
--website="" \
--identity="" \
--from=wallet \
--gas=300000 \
--keyring-backend file

  
# if use another port --node "tcp://127.0.0.1:$$657"
  ``` 
##  Delegate stake
```
pointd tx staking delegate $Valoper 1000000apoint --from=wallet --chain-id=point_10687-1 --gas=auto
```
##  Withdraw reward with commision
```
pointd tx distribution withdraw-rewards $Valoper--from=wallet --commission --chain-id=point_10687-1 --gas=auto
```
##  Balance
```
pointd q bank balances $(pointd keys show wallet -a)
```
##  Reset
```
pointd tendermint unsafe-reset-all --home $HOME/.pointd --keep-addr-book
