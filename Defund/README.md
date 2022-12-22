# Defund Testnet (defund-private-3)

![image](https://www.defund.app/images/Defund-2.png)

## <a href="https://defund.app/">🌎 Website </a>
## <a href="https://discord.gg/bWZqS6xBcK">💎 Discord </a>
## <a href="https://defund.explorers.guru/">🚀 Explorer </a>

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
defundd version         #v0.2.1
```
```
#Initialize the node
defundd config keyring-backend test
defundd config chain-id defund-private-3
defundd init $Moniker-name --chain-id defund-private-3
```
```
#Download Genesis
wget -O defund-private-3-gensis.tar.gz https://github.com/defund-labs/testnet/raw/main/defund-private-3/defund-private-3-gensis.tar.gz
sudo tar -xvzf defund-private-3-gensis.tar.gz -C $HOME/.defund/config
rm defund-private-3-gensis.tar.gz
sha256sum $HOME/.defund/config/genesis.json          # 1a10121467576ab6f633a14f82d98f0c39ab7949102a77ab6478b2b2110109e3
```
```
