# Cosmos Hub Mainnet (cosmoshub-4)

![image](https://cosmos.network/og-image.jpg)

## <a href="https://cosmos.network/">🌎 Website </a>
## <a href="https://discord.gg/cosmosnetwork">💎 Discord </a>
## <a href="https://www.mintscan.io/cosmos">🚀 Explorer </a>

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
git clone https://github.com/cosmos/gaia cosmos
cd cosmos
git checkout v7.1.0
make install
gaia version
# v7.1.0
```
## Initialize the node
```
gaiad config keyring-backend test
gaiad init Moniker --chain-id cosmoshub-4
```

## Download Genesis
```
wget -O $HOME/.gaia/config/genesis.json "https://raw.githubusercontent.com/Alexmed911/Nodes-Setup-Manuals/main/Cosmos/genesis.json"
sha256sum $HOME/.gaia/config/genesis.json
# 6ad715c1ab5637e505e7248bb4366e79d5dec1a24f4fac7db33fead567041633
```
## Create/recover wallet
```
gaiad keys add wallet 
gaiad keys add wallet --recover
```

## Configure Peers/Gas-prices/Indexing
```
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.00uatom"|g' $HOME/.gaia/config/app.toml
peers="f40a6e7d7168a3f2a5362cd37cbe6eac7a686056@185.229.119.178:26656,b858ca4f3fed2c36b949cf67188b126e2542a39a@135.181.215.115:26726,c14d39422b5d70d9084d19d286c7427c0762cdfc@162.55.92.114:2010,e829d4764a5cecc44b3414777853b34407b36601@185.16.39.179:26656,c62900f5d5b4f5ce9422e4ba123d637ea2fa6375@65.108.232.181:26656,dee3771d222681139d9df18d4e127d4f52820614@65.108.142.81:26656,3c08dde641866bf332e1a9f94261a39d57247328@65.108.230.27:26656,c9bebf70be2b703c9ee3e7fc97d2efbca7a84ed8@3.238.175.107:26656,73c2a86cc0d4b51c81bd0e36cee69f1731bcda0d@23.88.69.157:26656,847e0bf54b315e633a6d990de66a4c9721ba1830@206.189.26.213:26090,a0aca8fb801c69653a290bd44872e8457f8b0982@47.99.180.54:26656,3da88430414ec9084c8983fe4d462cce655ff1f3@51.222.245.114:26656,344d87e04fdf04be760da5069a59d9a489b886a6@52.14.44.1:26656,1cce99042f884d669e7287e3e362bff8e385c63e@46.4.79.183:26726"
sed -i -e 's|^seeds *=.*|seeds = "'$seeds'"|; s|^persistent_peers *=.*|persistent_peers = "'$peers'"|' $HOME/.gaia/config/config.toml
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.gaia/config/config.toml
```
## Download Addrbook
```
wget -O $HOME/.gaia/config/addrbook.json "https://raw.githubusercontent.com/Alexmed911/Nodes-Setup-Manuals/main/Cosmos/addrbook.json"
```
## Pruning
```
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.gaia/config/config.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.gaia/config/config.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.gaia/config/config.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruninaltheag_interval\"/" $HOME/.gaia/config/config.toml
```
## Create Service
```
sudo tee /etc/systemd/system/gaiad.service > /dev/null <<EOF
[Unit]
Description=Cosmos
After=network-online.target

[Service]
User=$USER
ExecStart=$(which gaiad) start
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
sudo systemctl enable gaiad
sudo systemctl restart gaiad && sudo journalctl -u gaiad -f -o cat
```
## State-Sync
```
cp $HOME/.gaia/data/priv_validator_state.json $HOME/.gaia/priv_validator_state.json.backup
gaiad tendermint unsafe-reset-all --home $HOME/.gaia --keep-addr-book
SNAP_RPC="https://cosmos-rpc.polkachu.com:443"
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 500)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.gaia/config/config.toml

mv $HOME/.gaia/priv_validator_state.json.backup $HOME/.gaia/data/priv_validator_state.json
sudo systemctl restart gaiad && journalctl -u gaiad -f -o cat
```
## Create validator
```
gaiad tx staking create-validator \
--amount=1000000uatom \
--pubkey=$(gaiad tendermint show-validator) \
--moniker=Name \
--chain-id=cosmoshub-4 \
--commission-rate="0.10" \
--commission-max-rate="0.20" \
--commission-max-change-rate="0.1" \
--min-self-delegation="1" \
--fees 500uatom \
--from=wallet \
--identity="" \
--website="" \
--details="" \
-y 

  
# if use another port --node "tcp://127.0.0.1:$$657"
  ``` 
##  Delegate stake
```
gaiad tx staking delegate $Valoper 1000000uatom --from=wallet --fees=300uatom --chain-id=cosmoshub-4
```
##  Withdraw reward with commision
```
gaiad tx distribution withdraw-rewards $Valoper --from=wallet --commission --fees=300uatom --chain-id=cosmoshub-4
```
##  Balance
```
gaiad q bank balances $(gaiad keys show wallet -a)
```
##  Reset
```
gaiad tendermint unsafe-reset-all --home $HOME/.gaia --keep-addr-book
