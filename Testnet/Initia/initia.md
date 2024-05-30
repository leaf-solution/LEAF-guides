# INITIA GUIDE 
MONIKER="LEAF-Yourmoniker"


## 1. UPDATE SYSTEM AND INSTALL BUILD TOOLS
```
sudo apt -q update
sudo apt -qy install curl git jq lz4 build-essential gcc unzip wget
```
after that reboot vps
```
reboot -f
```

upgrade
```
sudo apt -qy upgrade
```

after that reboot vps
```
reboot 

## 2. INSTALL GO

```
cd $HOME && \
ver="1.22.0" && \
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
rm "go$ver.linux-amd64.tar.gz" && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile && \
source ~/.bash_profile && \
go version
```

## 3. NODE INSTALLATION

### 3a. Build binary from source code

```
cd $HOME
rm -rf initia
git clone https://github.com/initia-labs/initia.git
cd initia
git checkout v0.2.15
make install
```
### 3b. Init 

```
initiad init "$MONIKER" --chain-id initation-1
```

### 3c. Download JSON Resource
```
wget -O $HOME/.initia/config/addrbook.json https://rpc-initia-testnet.trusted-point.com/addrbook.json
curl -Ls https://initia.s3.ap-southeast-1.amazonaws.com/initiation-1/genesis.json > \
         $HOME/.initia/config/genesis.json
```
### 3d. Config

```
sed -i \
  -e 's|^keyring-backend *=.*|keyring-backend = "test"|' \
  -e 's|^node *=.*|node = "tcp://localhost:13057"|' \
  $HOME/.initia/config/client.toml

sed -i 's|^pruning *=.*|pruning = "custom"|g' $HOME/.initia/config/app.toml
sed -i 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|g' $HOME/.initia/config/app.toml
sed -i 's|^pruning-interval *=.*|pruning-interval = "10"|g' $HOME/.initia/config/app.toml
sed -i 's|^snapshot-interval *=.*|snapshot-interval = 0|g' $HOME/.initia/config/app.toml

sed -i -e 's/external_address = \"\"/external_address = \"'$(curl httpbin.org/ip | jq -r .origin)':13056\"/g' ~/.initia/config/config.toml
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0.15uinit,0.01uusdc\"|" $HOME/.initia/config/app.toml
sed -i 's|^prometheus *=.*|prometheus = true|' $HOME/.initia/config/config.toml
sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:13058\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:13057\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:13060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:13056\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":13066\"%" $HOME/.initia/config/config.toml
sed -i -e "s%^address = \"tcp://localhost:1317\"%address = \"tcp://0.0.0.0:13017\"%; s%^address = \":8080\"%address = \":13080\"%; s%^address = \"localhost:9090\"%address = \"0.0.0.0:13090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:13091\"%; s%:8545%:13045%; s%:8546%:13046%; s%:6065%:13065%" $HOME/.initia/config/app.toml
sed -i -e "s/^max_num_inbound_peers *=.*/max_num_inbound_peers = 300/" $HOME/.initia/config/config.toml
sed -i -e "s/^max_num_outbound_peers *=.*/max_num_outbound_peers = 150/" $HOME/.initia/config/config.toml
```
### 3e. Config PEER
```
SEEDS=""
PEERS="3194727c8195c5819093b677a982be0d512fa033@89.187.191.103:26656,85b96d58f865afc7644e9d2cf0b6232503e6e1aa@18.159.7.240:26656,66ee7ad2493269340aa25da23e92dae136e3e7b4@158.220.124.150:26656,90a735bf21ee4364f3d021958d67915df52b431a@49.12.172.92:26656,f3b94684ee056875523f0a85d96325fc78d8d709@162.55.24.104:26656,59f20fef9b2451ce1d74501e3988eaeecb37c1f1@34.133.135.35:26656,57490a7e1a1c913ee1868824663974af6a9d3790@144.91.85.45:14656,8e24f9b3bc00ec82bab844b74d673b04c3e24123@162.55.199.187:26656,468b1a84c54628bd1a56bbcf4bfaa694384db79e@109.199.118.36:33756,a98e47c02763d05f2a9623cb67e8e40e0d06504a@5.9.70.180:15674,9488681d0886eed1d78f20c85a89f3d73d87aeee@84.247.166.190:26656,66abd758f6971eb8227fc54d11cb56ca1ca280e6@65.109.113.251:13656,d895ecbb679be0a4904d225abeb2d7d85078c543@65.108.232.248:26656,2f72cacc2038f22b7652f3c75e52e285f9e9b801@158.220.103.170:26656,9f58c18bf77af3c9c561217d6beb3acb2f1b067c@95.70.184.178:11656,6a6d164766341e4e4f56d0359f130a757f21851a@95.217.148.179:29656,3fd594106dc09070b2964d054eab33665433fb59@109.199.96.227:14656,548e26b95b895efc964b08a6b2e991c6d5a6791d@142.132.151.35:15674,697cd70147c7fcd16cf00ad39fe1217a35e2db15@38.58.183.3:31656,0845e0ee92cfd31e20e994a3ee1e9250571ea70e@38.242.135.3:14656,17c23f7fad1a74cc22966bae79d17fb62d7e4b00@207.180.228.81:26656,745e4f73e0d77d34dab6f2e2cbac96ac7fc15786@85.190.242.204:26656,cd69bcb00a6ecc1ba2b4a3465de4d4dd3e0a3db1@65.108.231.124:51656,17b0fb616bae3fc2e6babf717e2ec132353142db@51.195.88.136:15674,6fdfe2a83beaf7a120660a7a467df27e69439220@188.166.17.88:11656,87f7bcd688e0663b2f68c37b7c5c532fdbd4d25c@62.171.172.170:26656,8f6b3b8c519e98eeb31a0c92e771e1e51488a79d@116.202.140.75:26656,b2d980cf7b48fde5edda8331b45da1ab6ad32d17@38.58.183.44:51656,8c7ce3feb81910b059c0cedff0cca2934a91db85@81.17.101.103:53456,b7b9f62cd9234ad7a2bbd04723874855c78b3517@43.134.77.108:26656"
sed -i 's|^seeds *=.*|seeds = "'$SEEDS'"|; s|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.initia/config/config.toml
```
### 3f. Create service

```
sudo tee /etc/systemd/system/initia.service > /dev/null << EOF
[Unit]
Description=initia node
After=network-online.target
[Service]
User=$USER
ExecStart=$(which initiad) start
Restart=on-failure
RestartSec=10
LimitNOFILE=10000
[Install]
WantedBy=multi-user.target
EOF
```
### 3g. Start service
```
sudo systemctl daemon-reload
sudo systemctl enable initia
sudo systemctl start initia
sudo journalctl -fu initia -o cat
```

# SNAPSHOT

```
sudo systemctl stop initia
cp $HOME/.initia/data/priv_validator_state.json $HOME/.initia/priv_validator_state.json.backup
rm -rf $HOME/.initia/data && mkdir -p $HOME/.initia/data
sudo apt-get install aria2
aria2c -x 16 -s 16 -o initia.tar.lz4 http://snapshots.staking4all.org/testnet-snapshots/initia/latest/initia.tar.lz4
lz4 -dc initia.tar.lz4 | tar -xf - -C $HOME/.initia
mv $HOME/.initia/priv_validator_state.json.backup $HOME/.initia/data/priv_validator_state.json
sudo systemctl restart initia
journalctl -u initia -f
```

# BACKUP VALIDATOR (VERY IMPORTANT)

```
mkdir -p $HOME/backup/.initia
cp $HOME/.initia/config/priv_validator_key.json $HOME/backup/.initia
```


# USEFULL COMMAND

### MANAGE KEYS
Generate new key
```
initiad keys add wallet
```
Recover key
```
initiad keys add wallet --recover
```
... UPDATE LATER

### MANAGE VALIDATOR

```
initiad tx mstaking create-validator \
  --amount=1000000uinit \
  --pubkey=$(initiad tendermint show-validator) \
  --moniker=$MONIKER \
  --chain-id=initiation-1 \
  --identity="" \
  --website="https://leafsolution.io/" \
  --details="Easier staking, greater rewards" \
  --security-contact="email-address" \
  --commission-rate="0.05" \
  --commission-max-rate="0.20" \
  --commission-max-change-rate="0.05" \
  --gas=auto \
  --gas-adjustment=1.4 \
  --fees=300000uinit \
  --node https://rpc-initia-testnet.trusted-point.com:443 \
  --from=wallet \
  -y
```
... UPDATE LATER

# HOW TO GET PUBLIC RPC

Get IP and port
```
RPC="http://$(wget -qO- eth0.me)$(grep -A 3 "\[rpc\]" $HOME/.initia/config/config.toml | egrep -o ":[0-9]+")" && echo $RPC
```
Change config to public route
```
sed -i '/\[rpc\]/,/\[/{s/^laddr = "tcp:\/\/127\.0\.0\.1:/laddr = "tcp:\/\/0.0.0.0:/}' $HOME/.initia/config/config.toml
```
Restart node
```
sudo systemctl stop initia
sudo systemctl restart initia
```
Test, must response an json
```
curl $RPC/status
```
