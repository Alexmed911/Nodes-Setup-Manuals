Shardeum Testnet 

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
