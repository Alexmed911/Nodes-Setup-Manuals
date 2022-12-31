# Boostrom Mainnet (bostrom)

<a href="https://freeimage.host/i/HufbKUG"><img src="https://iili.io/HufbKUG.md.jpg" alt="HufbKUG.md.jpg" border="0"></a>

## <a href="https://cyb.ai/">🌎 Website </a>
## <a href="https://t.me/cyber">💎 Telegram </a>
## <a href="https://ping.pub/bostrom">🚀 Explorer </a>

# Manual Setup

## Install dependencies
```
sudo apt update && sudo apt upgrade -y
sudo apt install curl tar unzip wget tmux clang lz4 pkg-config libssl-dev jq build-essential git make ncdu gcc jq chrony liblz4-tool -y
```
## Install Docker
```
apt install apt-transport-https ca-certificates software-properties-common curl
curl -f -s -S -L https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable"
apt update
cd /root
sudo apt install docker-ce docker-ce-cli containerd.io docker-compose-plugin -y
sudo systemctl status docker  
```
## Install Nvidia drivers

```
sudo add-apt-repository ppa:graphics-drivers/ppa
sudo apt update
sudo apt install -y ubuntu-drivers-common
ubuntu-drivers devices

#You should see something similar to this:

== /sys/devices/pci0000:00/0000:00:01.0/0000:01:00.0 ==
modalias : pci:v000010DEd00001BA1sv00001462sd000011E4bc03sc00i00
vendor   : NVIDIA Corporation
model    : GP104M [GeForce GTX 1070 Mobile]
driver   : nvidia-driver-418 - third-party free
driver   : nvidia-driver-430 - third-party free
driver   : nvidia-driver-440 - third-party free
driver   : nvidia-driver-460 - third-party free recommended
driver   : xserver-xorg-video-nouveau - distro free builtin

sudo ubuntu-drivers autoinstall

#Reboot the system
sudo reboot

#Check drivers
nvidia-smi
```
## Install Nvidia container runtime for docker
```
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list
sudo apt-get update && sudo apt-get install -y nvidia-container-toolkit
sudo systemctl restart docker

#Test nvidia-smi with the latest official CUDA image
docker run --gpus all nvidia/cuda:11.4.0-base nvidia-smi
```
## Start Node 
```
mkdir $HOME/.cyber
mkdir $HOME/.cyber/data
mkdir $HOME/.cyber/config

docker run -d --gpus all --name=bostrom --restart always -p 26656:26656 -p 26657:26657 -p 1317:1317 -e ALLOW_SEARCH=false -v $HOME/.cyber:/root/.cyber  cyberd/bostrom:dragonberry-cuda11.4
```
## Create/recover wallet
```
docker exec -ti bostrom cyber keys add Name
docker exec -ti bostrom cyber keys add Name --recover
```

## Configure Peers/Gas-prices/Indexing
```
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.01boot"|g' $HOME/.cyber/config/app.toml
peers="5d542c0eb40ae48dc2cac0c140aedb605ded77dc@195.201.105.229:26656,d57fe08e781b3c3d89cb57e5060468f590793258@65.109.70.20:26656,d0518ce9881a4b0c5872e5e9b7c4ea8d760dad3f@85.10.207.173:26656,df79a86dc236b8dff250c402e95cd6853e8ad7c4@46.166.165.4:26656,39a20a7d84c6e91c6638f5a685a13f655e050ee0@176.37.214.146:26656,5e8522bef5ceca507e05aa0d5f67f37a70222c73@88.218.191.79:26656"
sed -i -e 's|^seeds *=.*|seeds = "'$seeds'"|; s|^persistent_peers *=.*|persistent_peers = "'$peers'"|' $HOME/.cyber/config/config.toml
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.cyber/config/config.toml
```
## Download Addrbook
```
wget -O $HOME/.cyber/config/addrbook.json "https://raw.githubusercontent.com/Alexmed911/Nodes-Setup-Manuals/main/Cyber/addrbook.json"
```
## Pruning
```
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.cyber/config/config.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.cyber/config/config.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.cyber/config/config.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.cyber/config/config.toml
```
## Snapshot
```
#Visit and download actual pruned snapshot
https://jupiter.cybernode.ai/shared/

wget https://jupiter.cybernode.ai/shared/bostrom_pruned_6161577_2022-12-26.tar
tar -xvf bostrom_pruned_6161577_2022-12-26.tar -C $HOME/.cyber

```
## Create validator
```
docker exec -ti bostrom cyber tx staking create-validator \
  --amount=100000000boot \
  --min-self-delegation "1000000" \
  --details "" \
  --website="" \
  --identity="" \
  --pubkey=$(docker exec -ti bostrom cyber tendermint show-validator) \
  --moniker="Name" \
  --from=Name \
  --commission-rate="0.10" \
  --commission-max-rate="0.20" \
  --commission-max-change-rate="0.01" \
  --chain-id=bostrom \
  --gas-prices 0.01boot \
  --gas 600000
  
# if use another port --node "tcp://127.0.0.1:$$657"
  ``` 
##  Delegate stake
```
docker exec -ti bostrom cyber tx staking delegate boostromvaloper 9999999991000boot --from=wallet --chain-id=boostrom --gas=300000 --keyring-backend file --fees=200boot
```
##  Unjail
```
docker exec -ti bostrom cyber tx slashing unjail --from=Name --chain-id bostrom --gas-prices 0.01boot --gas 300000
```
##  Reset
```
cyber tendermint unsafe-reset-all --home root/.cyber
