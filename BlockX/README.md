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
