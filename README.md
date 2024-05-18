# Initia Validator Node Setup Guide
![just a picture](https://github.com/zhizhi1348/initia-validator/blob/main/iamge.png)

## Official Links
[Initia Discord](https://discord.com/invite/initia)

[My Twitter](https://x.com/zhiyarrr1) for further updates

## Server Specs
```
CPU: 4 cores


Memory: 16GB RAM


Disk: 1 TB SSD Storage


Bandwidth: 100 Mbps
```

## Install Dependecies

```ruby
sudo apt update && sudo apt upgrade -y

sudo apt install curl git jq build-essential gcc unzip wget lz4 -y
```

## Install GO

```ruby
cd $HOME && \
ver="1.22.0" && \
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
rm "go$ver.linux-amd64.tar.gz" && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile && \
source $HOME/.bash_profile && \
go version
```

## Install Initia

```ruby
git clone https://github.com/initia-labs/initia
cd initia
git checkout v0.2.14
make build
```

```
mkdir -p $HOME/.initia/cosmovisor/genesis/bin
mv /root/initia/build/initiad $HOME/.initia/cosmovisor/genesis/bin/
```

## Link Genesis

```ruby
sudo ln -s $HOME/.initia/cosmovisor/genesis $HOME/.initia/cosmovisor/current -f
sudo ln -s $HOME/.initia/cosmovisor/current/bin/initiad /usr/local/bin/initiad -f
```

## Install Cosmovisor
```
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.5.0
```

## Create service file
```ruby
sudo tee /etc/systemd/system/initiad.service > /dev/null << EOF
[Unit]
Description=initia node service
After=network-online.target

[Service]
User=$USER
ExecStart=$(which cosmovisor) run start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
Environment="DAEMON_HOME=$HOME/.initia"
Environment="DAEMON_NAME=initiad"
Environment="UNSAFE_SKIP_BACKUP=true"
Environment="PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:$HOME/.initia/cosmovisor/current/bin"

[Install]
WantedBy=multi-user.target
EOF
```

```
sudo systemctl daemon-reload
sudo systemctl enable initiad.service
```

## Set Variables

***Replace your node name***

```ruby
echo 'export MONIKER="NODE_NAME"' >> ~/.bash_profile
echo 'export WALLET="wallet"' >> ~/.bash_profile
source $HOME/.bash_profile
```

## Initiate Node

```ruby
initiad config set client chain-id initiation-1
initiad config set client node tcp://localhost:15657
initiad config set client keyring-backend test
```

```
initiad init $MONIKER --chain-id initiation-1
```

## Genesis
```ruby
rm ~/.initia/config/genesis.json
curl -Ls https://raw.githubusercontent.com/molla202/pokemon/main/genesis.json > $HOME/.initia/config/genesis.json
curl -Ls https://raw.githubusercontent.com/Core-Node-Team/Testnet-TR/main/Initia/addrbook.json > $HOME/.initia/config/addrbook.json
```

## Set Ports

**You can replace "15" if you have port conflicts or leave it alone**

```ruby
echo "export N_PORT="15"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

## Add ports to app.toml & config.toml files

```ruby
sed -i.bak -e "s%:1317%:${N_PORT}317%g;
s%:8080%:${N_PORT}080%g;
s%:9090%:${N_PORT}090%g;
s%:9091%:${N_PORT}091%g;
s%:8545%:${N_PORT}545%g;
s%:8546%:${N_PORT}546%g;
s%:6065%:${N_PORT}065%g" $HOME/.initia/config/app.toml
```

```ruby
sed -i.bak -e "s%:26658%:${N_PORT}658%g;
s%:26657%:${N_PORT}657%g;
s%:6060%:${N_PORT}060%g;
s%:26656%:${N_PORT}656%g;
s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${N_PORT}656\"%;
s%:26660%:${N_PORT}660%g" $HOME/.initia/config/config.toml
```

## Peers

```ruby
PEERS="a3660a8b7a0d88b12506787b26952930f1774fc2@65.21.69.53:48656,e3ac92ce5b790c76ce07c5fa3b257d83a517f2f6@178.18.251.146:30656,2692225700832eb9b46c7b3fc6e4dea2ec044a78@34.126.156.141:26656,2a574706e4a1eba0e5e46733c232849778faf93b@84.247.137.184:53456,40d3f977d97d3c02bd5835070cc139f289e774da@168.119.10.134:26313,1f6633bc18eb06b6c0cab97d72c585a6d7a207bc@65.109.59.22:25756,4a988797d8d8473888640b76d7d238b86ce84a2c@23.158.24.168:26656,e3679e68616b2cd66908c460d0371ac3ed7795aa@176.34.17.102:26656,d2a8a00cd5c4431deb899bc39a057b8d8695be9e@138.201.37.195:53456,329227cf8632240914511faa9b43050a34aa863e@43.131.13.84:26656,517c8e70f2a20b8a3179a30fe6eb3ad80c407c07@37.60.231.212:26656,07632ab562028c3394ee8e78823069bfc8de7b4c@37.27.52.25:19656,028999a1696b45863ff84df12ebf2aebc5d40c2d@37.27.48.77:26656,3c44f7dbb473fee6d6e5471f22fa8d8095bd3969@185.219.142.137:53456,8db320e665dbe123af20c4a5c667a17dc146f4d0@51.75.144.149:26656,c424044f3249e73c050a7b45eb6561b52d0db456@158.220.124.183:53456,767fdcfdb0998209834b929c59a2b57d474cc496@207.148.114.112:26656,edcc2c7098c42ee348e50ac2242ff897f51405e9@65.109.34.205:36656,140c332230ac19f118e5882deaf00906a1dba467@185.219.142.119:53456,4eb031b59bd0210481390eefc656c916d47e7872@37.60.248.151:53456,ff9dbc6bb53227ef94dc75ab1ddcaeb2404e1b0b@178.170.47.171:26656,ffb9874da3e0ead65ad62ac2b569122f085c0774@149.28.134.228:26656" && \
SEEDS="2eaa272622d1ba6796100ab39f58c75d458b9dbc@34.142.181.82:26656,c28827cb96c14c905b127b92065a3fb4cd77d7f6@testnet-seeds.whispernode.com:25756" && \
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.initia/config/config.toml
```

## set gas

```ruby
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0.15uinit,0.01uusdc\"|" $HOME/.initia/config/app.toml
```

## Prunning
```ruby
sed -i -e "s/^pruning *=.*/pruning = \"custom\"/" $HOME/.initia/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"100\"/" $HOME/.initia/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"50\"/" $HOME/.initia/config/app.toml
```

## Download Snapshot of a synced node files

```ruby
sudo systemctl stop initiad
```

```ruby
initiad tendermint unsafe-reset-all --home $HOME/.initia --keep-addr-book
```

```ruby
wget https://rpc-initia-testnet.trusted-point.com/latest_snapshot.tar.lz4
```

```ruby
lz4 -d -c ./latest_snapshot.tar.lz4 | tar -xf - -C $HOME/.initia
```

## Start Node
```ruby
sudo systemctl daemon-reload && \
sudo systemctl enable initiad && \
sudo systemctl restart initiad
```

## Check Logs

```
sudo journalctl -u initiad -f -o cat
```

***Ctrl + C to exit***

## Sync Check
**True = Node is NOT synced**
**False = Node is synced**
```ruby
initiad status | jq
```
***Only Continue the steps if you are getting FALSE***

## Create/Recover Wallet
**Create Wallet**
```ruby
initiad keys add $WALLET
```

**Recover Wallet**
```ruby
initiad keys add wallet --recover
```

## Get Faucet
***Get [Faucet](https://faucet.testnet.initia.xyz) for your node wallet***

## Check Balance
```ruby
initiad q bank balances $(initiad keys show $WALLET -a)
```

**You can import your seed phrase to initia wallet extension to send tokens or check balance too**

## Create Validator
```ruby
initiad tx mstaking create-validator \
  --amount=5000000uinit \
  --pubkey=$(initiad tendermint show-validator) \
  --moniker=$MONIKER \
  --chain-id=initiation-1 \
  --commission-rate=0.05 \
  --commission-max-rate=0.10 \
  --commission-max-change-rate=0.01 \
  --from=$WALLET \
  --identity="" \
  --website="https://github.com/0xMoei" \
  --details="0xMoei Validator" \
  --node=http://localhost:15657 \
  --gas-adjustment 1.4 \
  --gas auto \
  --gas-prices 0.15uinit \
  -y
```

**Save the txhash**

## Delegate Tokens to your Validator
```ruby
initiad tx mstaking delegate $(initiad keys show $WALLET --bech val -a)  1000000uinit --from $WALLET --gas-adjustment 1.4 --gas auto --gas-prices 0.15uinit --node=http://localhost:15657 -y
```
## WELL DONE! YOU JUST RAN YOUR VALIDATOR NODE ON INITIA TESTNET!

## Cheat sheet

**Check inActive Validators (search your node name)**
```
initiad q mstaking validators -o json --limit=1000 \
| jq '.validators[] | select(.status=="BOND_STATUS_UNBONDED")' \
| jq -r '.voting_power + " - " + .description.moniker' \
| sort -gr | nl
```
**Check Active Valdiators**
```
initiad q mstaking validators -o json --limit=1000 \
| jq '.validators[] | select(.status=="BOND_STATUS_BONDED")' \
| jq -r '.voting_power + " - " + .description.moniker' \
| sort -gr | nl
```
**Validator Status**
```
initiad q mstaking validator $(initiad keys show $WALLET --bech val -a)
```
**Check Logs**
```
sudo journalctl -u initiad -f -o cat
```
**Restart Node**
```
sudo systemctl restart initiad
```
**Stop Node**
```
sudo systemctl stop initiad
```
**Unjail Validator**
```
initiad tx slashing unjail --from $WALLET --fees=0.025uinit -y
```
