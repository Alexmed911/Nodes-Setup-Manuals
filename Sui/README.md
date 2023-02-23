# Sui Devnet/Testnet

![image](https://storage.googleapis.com/sui-cms-content/Sui_Dev_Net_42f85c7968/Sui_Dev_Net_42f85c7968.png)

## <a href="https://sui.io/">🌎 Website </a>
## <a href="https://discord.gg/sui">💎 Discord </a>
## <a href="https://www.scale3labs.com/check/sui">🚀 Checker </a>

# Manual Setup

## Install dependencies
```
sudo apt update && sudo apt upgrade -y
sudo apt install curl tar unzip wget tmux clang lz4 pkg-config libssl-dev jq build-essential git make ncdu gcc jq chrony liblz4-tool libclang-dev libpq-dev cmake -y
```
## Install Rust
```
sudo curl https://sh.rustup.rs -sSf | sh -s -- -y
source $HOME/.cargo/env
rustc --version
```
## Install Node

```
cd $HOME
git clone https://github.com/MystenLabs/sui.git
cd sui
git remote add upstream https://github.com/MystenLabs/sui
git fetch upstream
git checkout -B testnet --track upstream/testnet // or git checkout -B devnet --track upstream/devnet
mkdir $HOME/.sui
```

## Download Genesis
```
# Devnet
wget -qO $HOME/.sui/genesis.blob https://github.com/MystenLabs/sui-genesis/raw/main/devnet/genesis.blob

# Testnet
wget -qO $HOME/.sui/genesis.blob https://github.com/MystenLabs/sui-genesis/blob/main/testnet/genesis.blob?raw=true
```
## Config
```
cp $HOME/sui/crates/sui-config/data/fullnode-template.yaml \
$HOME/.sui/fullnode.yaml

sed -i -e "s%db-path:.*%db-path: \"$HOME/.sui/db\"%; "\
"s%metrics-address:.*%metrics-address: \"0.0.0.0:9184\"%; "\
"s%json-rpc-address:.*%json-rpc-address: \"0.0.0.0:9000\"%; "\
"s%genesis-file-location:.*%genesis-file-location: \"$HOME/.sui/genesis.blob\"%; " $HOME/.sui/fullnode.yaml
```
## Create Service
```
printf "[Unit]
Description=Sui node
After=network-online.target

[Service]
User=$USER
ExecStart=`which sui-node` --config-path $HOME/.sui/fullnode.yaml
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target" > /etc/systemd/system/suid.service
```
## Start Node Service
```
sudo systemctl daemon-reload
sudo systemctl enable suid
sudo systemctl restart suid && sudo journalctl -u suid -f -o cat
```
##  Create wallet
```
sui client 
#y
#Enter
#0
