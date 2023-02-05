# Shardeum Betanet 

![image](https://shardeum.org/Shardeum.png)

## <a href="https://shardeum.org/">🌎 Website </a>
## <a href="https://discord.gg/shardeum">💎 Discord </a>
## <a href="https://explorer-sphinx.shardeum.org/">🚀 Explorer </a>

# Manual Setup

## Install dependencies
```
sudo apt update && sudo apt upgrade -y
sudo apt install curl tar unzip wget tmux clang build-essential git make ncdu gcc jq -y
```
## Install Docker
```
sudo apt install docker.io
docker --version
# 20.10.23, build 7155243
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
docker-compose --version
#docker-compose version 1.29.2, build 5becea4c
```
## Install Node 

```
curl -O https://gitlab.com/shardeum/validator/dashboard/-/raw/main/installer.sh && chmod +x installer.sh && ./installer.sh

#The terminal will ask questions about your setup settings : password/port/directory

```
## Validator CLI
```
cd .shardeum
./shell.sh
operator-cli gui start
#Go to your web browser and go to:
https://your_node_ip:8080/

```
![image](https://docs.shardeum.org/assets/images/loginPage-908fe8d97a77c39e92c16d8fe73c7cdd.png)

# You should see the “Overview” page for the Shardeum Validator Dashboard in your web browser:

![image](https://docs.shardeum.org/assets/images/overviewBetanet-4ffa4b2b726131cca036f002391e06f3.png)

# Go to the “Maintenance” page, then click the “Start Node” button in the top left white box:

![image](https://docs.shardeum.org/assets/images/startBetanet-3887761f685e3e0c2785dc609f7db4df.png)

# Connect Wallet to Betanet https://docs.shardeum.org/Network/Endpoints#connect-wallet

![image](https://docs.shardeum.org/assets/images/connectWalletBetanet-16d8f440bb744e8946309acfe5270219.png)

# Use Faucet 

# https://docs.shardeum.org/Faucet/Claim#shardeum-faucet-website

# Add Stake 

![image](https://docs.shardeum.org/assets/images/connectedWalletAddStake-cb4ad52d4df8267630fb41cf8397f28d.png)

## If your node status is on Standby and you have 10 SHM or more staked, your validator node is setup correctly.

## The network will automatically add your validator to be active in the network.

## The time to be added as an active validator will vary based on network load and validators in the network.
