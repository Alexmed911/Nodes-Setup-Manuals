# Nibiru Testnet (nibiru-testnet-2)

![image](https://techcrunch.com/wp-content/uploads/2022/09/nibiru.jpg?resize=1200,587)

## <a href="https://nibiru.fi/">🌎 Website </a>
## <a href="https://discord.gg/dx7WTtuRNR">💎 Discord </a>
## <a href="https://nibiru.explorers.guru/">🚀 Explorer </a>

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
git clone https://github.com/NibiruChain/nibiru
cd nibiru
git checkout v0.16.3
make install
nibid version         
#v0.16.3
```
## Initialize the node
```
nibid init Name --chain-id=nibiru-testnet-2
```

## Download Genesis
```
wget -O $HOME/.nibid/config/genesis.json "https://raw.githubusercontent.com/Alexmed911/Nodes-Setup-Manuals/main/Nibiru/genesis.json"
sha256sum $HOME/.nibid/config/genesis.json
# 5cedb9237c6d807a89468268071647649e90b40ac8cd6d1ded8a72323144880d
```
## Create/recover wallet
```
nibid keys add key_name
nibid keys add key_name --recover
```

## Configure Peers/Gas-prices/Indexing
```
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.001unibi"|g' $HOME/.nibid/config/app.toml
peers="62f26443c930a02f3e166b9db4ecd37b65b042f2@49.12.8.255:26656,178f7dd47502283f9245d24ffcc0a0acc9f661cc@135.181.145.58:26656,0fc167f54fc0d63369763b1519e79c3b400c4bb4@65.108.97.58:2486,1a32af5ac53fc3180327c0045c492140e8b8bcb6@144.91.80.171:26656,bd0117a9200937887d854b14a7ea53f7ba2c81ea@185.245.183.192:46656,76d1973e958a340f7109a3d2bda6436b68f3bc6a@173.212.214.154:46656,888f65c496c3cb5ca345d227e01996506a52f65e@185.209.223.127:39656,bef7f536be357a2d69c643128c7f1c8245b76809@65.21.91.50:26656"
sed -i -e 's|^seeds *=.*|seeds = "'$seeds'"|; s|^persistent_peers *=.*|persistent_peers = "'$peers'"|' $HOME/.nibid/config/config.toml
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.nibid/config/config.toml
```
## Download Addrbook
```
wget -O $HOME/.nibid/config/addrbook.json "https://raw.githubusercontent.com/Alexmed911/Nodes-Setup-Manuals/main/Nibiru/addrbook.json"
```
## Pruning
```
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.nibid/config/config.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.nibid/config/config.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.nibid/config/config.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.nibid/config/config.toml
```
## Create Service
```
sudo tee /etc/systemd/system/nibid.service > /dev/null <<EOF
[Unit]
Description=Nibiru
After=network-online.target

[Service]
User=$USER
ExecStart=$(which nibid) start
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
sudo systemctl enable nibid
sudo systemctl restart nibid && sudo journalctl -u nibid -f -o cat
```
## Snapshot
```
cd $HOME/.nibid
sudo systemctl stop nibid
cp $HOME/.nibid/data/priv_validator_state.json $HOME/.nibid/priv_validator_state.json.backup
nibid tendermint unsafe-reset-all --home $HOME/.nibid --keep-addr-book
SNAP_NAME=$(curl -s https://snapshots.bccnodes.com/testnets/nibiru/ | egrep -o ">nibiru-testnet-2.*tar" | tail -n 1 | tr -d '>'); \
wget -O - https://snapshots.bccnodes.com/testnets/nibiru/${SNAP_NAME} | tar xf -
mv $HOME/.nibid/priv_validator_state.json.backup $HOME/.nibid/data/priv_validator_state.json
sudo systemctl restart nibid && journalctl -u nibid -f -o cat
nibid status 2>&1 | jq .SyncInfo
```
## Faucet
```
https://discord.com/channels/947911971515293759/984840062871175219
```
## Create validator
```
nibid tx staking create-validator \
--from wallet \
--amount 1000000unibi \
--pubkey "$(nibid tendermint show-validator)" \
--chain-id nibiru-testnet-2 \
--moniker="Name" \
--commission-max-change-rate=0.01 \
--commission-max-rate=1.0 \
--commission-rate=0.07 \
--min-self-delegation="1" \
--website="" \
--identity="" \
--details "" \
--security-contact="" \
--fees=5000unibi 
-y

  
# if use another port --node "tcp://127.0.0.1:$$657"
  ``` 
##  Delegate stake
```
nibid tx staking delegate $Valoper 10000000ulava --from=wallet --chain-id=nibiru-testnet-2 --gas=auto
```
##  Balance
```
nibid q bank balances $(nibid keys show wallet -a)
```
##  Reset
```
nibid tendermint unsafe-reset-all --home $HOME/.nibid --keep-addr-book
