# Persistence Mainnet (core-1)

![image](https://icodrops.com/wp-content/uploads/2020/09/Persistence_cover.jpeg)

## <a href="https://persistence.one/">🌎 Website </a>
## <a href="https://discord.gg/qXRmTTGcYD">💎 Discord </a>
## <a href="https://www.mintscan.io/persistence">🚀 Explorer </a>

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
git clone https://github.com/persistenceOne/persistenceCore.git
cd persistenceCore
git checkout v5.0.0
make install
persistenceCore version         
#v5.0.0
```
## Update v5
```
cd $HOME/persistenceCore
git pull
git checkout v5.0.0
make build
cp build/persistenceCore /root/go/bin
```
## Initialize the node
```
persistenceCore config keyring-backend test
persistenceCore init Name --chain-id core-1
```

## Download Genesis
```
wget -O $HOME/.persistenceCore/config/genesis.json "https://raw.githubusercontent.com/Alexmed911/Nodes-Setup-Manuals/main/Persistence/core-1/genesis.json"
sha256sum $HOME/.persistenceCore/config/genesis.json
# 66d0ffe2912176907e62e94bbec3b20b8ac4171723378e6054123f513482d782
```
## Create/recover wallet
```
persistenceCore keys add [key_name]
persistenceCore keys add [key_name] --recover
```

## Configure Peers/Gas-prices/Indexing/Seeds
```
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.0001uxprt"|g' $HOME/.persistenceCore/config/app.toml
peers="28f7c36d68e6be6a91742710a4bda6e773390133@35.215.35.133:26656,8e4e1f1e087c76c71c64e477e95495833da82aa2@95.217.65.53:26656,b9beb51b624f162d5923550ed605a1c966c17bed@65.108.106.244:20656,b3ebed629f25548665d6f855464784fdb2752ba6@65.108.98.235:28556,6d37cfef1c6fa294709eaef0d622d03ae479adad@185.158.177.235:31656,f2e5a7b9e54d2b0ac41e8367354e273ec1a9a40c@93.189.30.112:26656,e5d9a0744087ebbdef5eebc274326736424f261c@195.201.63.87:39666,7b9839cd3e994c44cbd747d1ddc51ee695f60e58@157.90.134.48:26656,ecc00c5a7abd057ea5ca4a94c48d1d937bbab34a@34.118.19.56:26656,e302b2bad4e22afa824fec6a31f5f9eeb968fbe3@217.76.53.18:26656,74d0da956485b22ae339b3c5709fea7b2e48ae67@146.59.81.92:33656,6d563fdd153d39b24ba3dee45c553825a2383999@167.235.2.68:2080,85081903e8d6dda9e7380c64f68f3f4febac5a09@52.13.244.141:26656,44839b3204b54c9bf73e2f89e25f6f38dd750871@13.234.77.52:26656,1fdf2607ef67817d695ea12c3c0ec5d84a1596b8@3.235.239.157:26656,3872a472e5e73e35b86a0a298e1be7f6465219df@52.78.123.16:26656,4143cf46326ce66a1d90fefa16673e5215919224@23.88.72.5:26194,d68cba170d40f7b36962a5e44048e11b55db8375@157.90.180.62:26657,f4b36ad9bb8d921a67a0521f8a75dcef1b73a631@152.32.186.207:26656,5a9ece73ed12cf04c39a482657b87f96c977e4f1@141.95.84.102:28656,53aeb694fb75d7018a25a766852cea898817e10c@65.108.3.247:26656,aee92f0900598f03a34a3cae87e76ebd0ba4fce4@65.108.140.219:26656,2b8e70fd5ace9ee226c18ee79a28f2916e9b739f@172.104.205.110:26656,cb024268fa57a8117a7412e1e847199f4f14d362@65.21.204.171:48656,98f9f130ae7d31273d38456da25327134dee2e6a@34.80.68.114:26656,1c880e0f615e743310f4619f6711e9d88448d3bd@54.169.213.13:26656,e3ccdce4ca45392f5385d692f6706ed0515070bb@13.250.3.119:26656,a00e0fc7e921c05e643d801e3f85b6e7f12a634b@212.227.190.180:26656,d7c62ecbe44772868ca1ecd9315c5f11a069da5b@85.215.101.214:26656,405f158dab7b18769bb62776d436e4a9a6144117@95.214.55.227:26896,ff2948823637ac389aa6db8516f22ef7ff85d199@141.94.211.135:26656,e640e521957974cbfb1c851aacdbc3115a606f39@3.108.249.47:26656,82588f011491c6100d922d133f52fc23460b9231@95.217.91.237:26656,ffee7b451409b1b3d6f0b08a68e9a6fcf9a094ca@52.66.222.184:26656,f73e1274545ee26324f9ef5766845fd37cfc8582@15.235.53.45:2042,c124ce0b508e8b9ed1c5b6957f362225659b5343@136.243.248.189:26656,1eba6535aa255b0cc9e7209da945729c357f6447@65.108.73.25:36656,ff12247f50ce7cdd9a4e8bea9d98c889653d12d0@159.89.101.239:26050,7f6f551c5f5a20b5e6768ce16b37ccf5c3ed2bab@135.181.213.223:26656,1aad2fd176088f66b305e770b811f6f9be645dc4@65.108.127.158:26657,99a8d750bd69e3c737205ee413fc4c43f3e473dd@13.235.27.46:26656,471518432477e31ea348af246c0b54095d41352c@88.198.131.125:26656,dfef569ebadced8fdb632cdbfc39807f3db5d9c0@23.88.71.91:26656,ea08f9323a540cbafc1efe572c76634d70a09ed7@13.250.231.113:26656,180d060cd6f19f4106108cea4079339f83a32687@152.67.3.239:36656,c306ab33a4f1eb3698396a6a7c33789bd82a2472@44.211.63.171:26656,c01bd9cfcc10a4781ca23012d61e454806c662cf@65.21.238.147:48656,4317f6547256b6ab75f13c80483179371c653224@204.93.241.113:26656,cc04b39c48ed7e66aa3ef745d23ab3dcd6907325@3.140.198.144:26656,cd8b8a33eea7656f3c3bbbfec93d3aab5a17e183@77.68.22.38:26656,f99cae0661913a87a1f1401451585b43724504da@95.216.202.37:26656,3b89355c69d2638526c5e9c07d42c49b2791b014@52.66.51.115:26656,2c4433a5398ed41845385d413a720e95d524a126@93.115.25.106:33656,f94b97e0f30450fc1a4d21b42dbcfcd10d4c54b4@34.159.153.131:26656,ca2ab57f9bd2c366956369e074957f109ccdfb29@178.154.192.10:26656,bba10290da32f3cb41e15c3a192413666ce05cee@5.9.208.13:26656,2ad7b01a76b568703410cffa7cb53c5a8b3a7080@209.126.84.180:26656,9755cab2585a2794453a5b396ef13b893393366f@65.108.212.224:46660,832a57b63024b567870195d14e2426d136acdd6b@65.108.4.188:15456,4b32259fb55686aa768db36d0253fde4c3e0c98d@94.250.201.145:26656,c375e881cd79e142079284bb417b5f19a7bdb999@47.242.65.248:26656,f116b1a17d4b26d0ff5fc2dba0009590061fbbe7@149.28.111.247:26656,51df969894f0650dbd7dd13f5cc604ef62fa0be1@54.194.229.95:26656,8a210f1bcfc9015a7bc18dcc5add29c0dce3f2dc@95.217.70.61:26656,53b64b05da4bf9424fd810ac1f0ef75b93f9e8aa@5.75.174.21:26756,97e4468ac589eac505a800411c635b14511a61bb@5.9.239.237:26656,7270b25b8c4f6fd184fc2f9be599addb1d16c231@3.113.33.9:26656,6cceba286b498d4a1931f85e35ea0fa433373057@88.198.128.173:26656,4e1c2471efb89239fb04a4b75f9f87177fd91d00@95.217.82.77:26656,cff05ceccb28780c156e853a1c89d46786febeea@3.23.92.224:26656,010e7673d46bfa3bf1f8cfcd7cd20bb4fc2e603d@3.123.1.56:26656,66cf9f3da2f1917b7b7c9af66e8e6df0266e0539@34.148.77.25:26656,837e463be94b52293a57dd3879ece0d54ef0227b@146.59.87.220:26656,bf21941ffd66d1c7cc12b70158dc79bf7310d08a@159.203.187.36:26050,254c59afc5c2a7f8e26447871faed3319caf95ff@51.161.87.113:26656,6b3644aba4149e7a4ea5ccf80bee3c5054c6d630@2.126.235.178:26656,61347e27a267cf7e6a2b639ba3688886ed9bb3cb@163.172.164.123:26656,aee49cc596cd8af39e4cb89754cb01cba4c157f7@52.40.69.129:26656,7dbc24f8992d2143598b8720f64b3b96a0f8f736@34.89.165.216:26656,a23857f73fe8507943b4eee038dc0d3955d242ad@51.79.177.229:26656,5e673932cd7e8269ff6a4915fcf0e17e15f20a68@89.163.133.19:26656,4f8cd6b250e7b7f5a1e385186246df7ad7de2ab0@212.95.53.155:26656,74b970832a772372bbe1794fcfbfc3be94608cae@65.109.2.226:26656,6774ffbe82d189a1ed1576c4dedf12a15722d448@157.245.148.115:26656,e6d16c94f512a5ad821ca223531fb462e3184faf@34.90.125.222:26656,4a32587b35886bb3f422f8f34436be503e39eb42@38.242.142.199:46656,c661f36cdbb9027405b98f9377a1d078d16880f0@51.91.83.94:2080,888b2697534b8e5dc4bbea72037c9de742e86684@95.217.85.254:15607,e576a910d07808240553f3f261235051d846d4f5@46.166.143.84:26656,0076c5ded42f96a1d37196c10be32dd4cc981ab2@135.181.217.163:26656,d4ff6ea6546aecd8cac0df76e4dd3838f3ab2d9f@15.207.18.221:26656,05412963529461b2c870c088057d8376e5a92ce7@139.144.29.29:26656,6ebe1c8855cc71bfa13a3d15426701a1a2ee1b6e@3.137.216.89:26656,e4be7155351af5aef8a25ad5a61a620b79495e08@51.38.53.101:26625,e0e1dd9d329d60c43b642f77dbce7199068fb2fa@3.6.37.52:26656,9047d858b8c71dc4e2e562ff2ba4a31e90f1bd38@107.135.15.67:26696,7cf539606cd3c188a93226c95670810a4ea5873f@80.64.211.45:26656,20103d05ce04ea66879b444e485b0ee91834d01e@51.195.145.107:26656,a08045bac640469f15fcfacf2182ebb519e02af9@46.20.245.41:26656,858f8a61f6eac359858215b4358d1675085f9d9e@35.201.129.255:26656,0393c19b176d1cf8bc560c5a8fa990301deb1a7e@95.216.235.53:26656,21379784b21109c8f1671b13879f92318dadc667@35.74.104.174:36650,a458cd95475dc19b9fad17bf6ccd57cd72144f13@128.199.128.15:26050,dbb22b2022d3a73f8f3f6a1f5225065c6ba9dd1d@185.144.83.138:26656,3823743e174b085dadab4b46454a9a5c1b2768d9@75.119.148.118:26656,21f3967bf650d076859a69fe234e42cba6815d46@73.117.148.143:26656,582fa26fe3d44ba963c620212d904308c25d2d01@157.90.90.77:16656,7cc92a9e3dcad37e5e7b3adf7814c37070fa9787@161.97.187.189:26656,646d0ad08c408f93276f90cd29d4e410e2d60f63@193.34.144.156:25656,022ef1087f40e26588d3d4fa7236a3252f87b2d2@65.108.99.37:26696,cd5b07c65242736eec707950280858267f3507ac@195.189.96.121:33656,80e34fa23b07bf01679a6dd4f645296622d3571a@37.120.245.178:26656,e50c644dc8f7f9acc183b5a990f184eb0bc7663b@35.77.189.232:26656,d68c851cff9abcddb6b660770c15c5f126beac53@176.9.188.21:38656,d5d5a7242f9ec0c68417d8921ab03caea08ccb03@178.62.23.83:26656,e7076e47e27198c6facb70e005c39ba9712d3fd7@207.154.224.146:26656,01c55a173325e0a024806807d38094fb7aa81310@162.55.234.70:54756,ebc272824924ea1a27ea3183dd0b9ba713494f83@178.211.139.77:26896,eb775b88a3bcedcb81293227669e0345d629cd2d@85.215.105.19:15607,cf2ae533d05a0ce7040d0cea43b2947bcc39894a@95.165.89.222:26565,fc6e77a255eaa46574e31d4ea25ab4c94d2b058b@3.239.100.132:26656,91d4802dfc07466e481d51a63150462125cf1800@65.108.122.246:26696,827e9b950ae345bdde22f17a510f8d5709d01a11@195.201.218.187:54656,d10ccca931c6ff64b29a2abbd4f7323da8325066@43.135.160.23:26656,4fb47f606460933398720f7644ef6611b85cc229@51.158.156.86:2080,b489c89a72804dc5f3b5589a7c35fb7045a5abe4@52.211.186.71:26656,fbf0aa94b512902a249b246ed5763b50df9c0543@178.128.221.238:26656,2c9e70a7bbdc61ad90eb6a540a7964cf0d7087b8@138.201.23.39:27656"
sed -i -e 's|^seeds *=.*|seeds = "'$seeds'"|; s|^persistent_peers *=.*|persistent_peers = "'$peers'"|' $HOME/.persistenceCore/config/config.toml
seeds="ade4d8bc8cbe014af6ebdf3cb7b1e9ad36f412c0@seeds.polkachu.com:15456"
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.persistenceCore/config/config.toml
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.persistenceCore/config/config.toml
```
## Download Addrbook
```
wget -O $HOME/.persistenceCore/config/addrbook.json "https://raw.githubusercontent.com/Alexmed911/Nodes-Setup-Manuals/main/Persistence/core-1/addrbook.json"
```
## Pruning
```
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.persistenceCore/config/config.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.persistenceCore/config/config.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.persistenceCore/config/config.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.persistenceCore/config/config.toml
```
## Create Service
```
sudo tee /etc/systemd/system/persistenceCore.service > /dev/null <<EOF
[Unit]
Description=persistenceCore
After=network-online.target

[Service]
User=$USER
ExecStart=$(which persistenceCore) start
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
sudo systemctl enable persistenceCore
sudo systemctl restart persistenceCore && sudo journalctl -u persistenceCore -f -o cat
```
## Snapshot
```
cd $HOME
sudo systemctl stop persistenceCore
cp $HOME/.persistenceCore/data/priv_validator_state.json $HOME/.persistenceCore/priv_validator_state.json.backup
rm -rf $HOME/.persistenceCore/data
SNAP_NAME=$(curl -s http://snapshots.autostake.net/core-1/ | egrep -o ">core-1.*.tar.lz4" | tr -d ">" | tail -1)
wget -O - http://snapshots.autostake.net/core-1/$SNAP_NAME | lz4 -dc - | tar -xf - -C  $HOME/.persistenceCore
mv $HOME/.persistenceCore/priv_validator_state.json.backup $HOME/.persistenceCore/data/priv_validator_state.json
sudo systemctl restart persistenceCore && journalctl -u persistenceCore -f -o cat
persistenceCore status 2>&1 | jq .SyncInfo
```
## Create validator
```
persistenceCore tx staking create-validator \
--from wallet \
--amount 1000000uxprt \
--pubkey "$(persistenceCore tendermint show-validator)" \
--chain-id core-1 \
--moniker="Name" \
--commission-max-change-rate=0.01 \
--commission-max-rate=1.0 \
--commission-rate=0.05 \
--min-self-delegation="1" \
--website="" \
--identity="" \
--details "" \
--security-contact="" \
--fees=30uxprt
  
   # if use another port --node "tcp://127.0.0.1:$$657"
  ``` 
##  Delegate stake
```
persistenceCore tx staking delegate $Valoper 2000000uxprt --from=wallet --chain-id=core-1 --gas=175000 --fees=40uxprt
```

