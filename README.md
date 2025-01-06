# Dymension
![1500x500](https://user-images.githubusercontent.com/91562185/234884978-f1a6b9f1-5939-422c-af5d-ca66a9feb758.jpg)

 * [Topluluk kanalımız](https://t.me/corenodechat)<br>
 * [Topluluk Twitter](https://twitter.com/corenodeHQ)<br>

## Sistem Gereksinimleri
| Bileşenler | Minimum Gereksinimler | 
| ------------ | ------------ |
| CPU |	8|
| RAM	| 32+ GB |
| Storage	| 500+ GB SSD |


# update ve kütüphane kuruyoruz
```
sudo apt update && sudo apt upgrade -y
sudo apt install curl git make wget htop tmux build-essential jq make lz4 gcc unzip -y  
```
# go kuruyoruz ( bazısında problem oluyor go version çıktı kontrol edin birden fazla go kurulumu ekledim )
```
wget -O go_install.sh https://raw.githubusercontent.com/Core-Node-Team/Testnet-TR/main/Dymension-mainnet/go_install.sh && chmod +x go_install.sh && ./go_install.sh
source ~/.bash_profile
go version
```
```
sudo rm -rvf /usr/local/go/
wget https://golang.org/dl/go1.22.7.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.22.7.linux-amd64.tar.gz
rm go1.19.3.linux-amd64.tar.gz
```
Not: go version çıktısı alamazsanız altakini deneyin.
```
cd $HOME
! [ -x "$(command -v go)" ] && {
VER="1.22.7"
wget "https://golang.org/dl/go$VER.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$VER.linux-amd64.tar.gz"
rm "go$VER.linux-amd64.tar.gz"
[ ! -f ~/.bash_profile ] && touch ~/.bash_profile
echo "export PATH=$PATH:/usr/local/go/bin:~/go/bin" >> ~/.bash_profile
source $HOME/.bash_profile
}
[ ! -d ~/go/bin ] && mkdir -p ~/go/bin
```

### Varyasyonlar
```
echo "export DYMENSION_PORT="205"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```
### Binary build
```
cd $HOME
rm -rf dymension
git clone https://github.com/dymensionxyz/dymension.git
cd dymension
git checkout v3.1.0
```
```
make build
```
```
mkdir -p $HOME/.dymension/cosmovisor/genesis/bin
mv build/dymd $HOME/.dymension/cosmovisor/genesis/bin/
rm -rf build
```
```
cd $HOME
rm -rf dymension
git clone https://github.com/dymensionxyz/dymension.git
cd dymension
git checkout v3.2.0
```
```
make build
```
```
mkdir -p $HOME/.dymension/cosmovisor/upgrades/v4/bin
mv build/dymd $HOME/.dymension/cosmovisor/upgrades/v4/bin/
```

### System Link
```
ln -s $HOME/.dymension/cosmovisor/genesis $HOME/.dymension/cosmovisor/current -f
sudo ln -s $HOME/.dymension/cosmovisor/current/bin/dymd /usr/local/bin/dymd -f
```
```
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.5.0
```
### Servis
```
sudo tee /etc/systemd/system/dymd.service > /dev/null << EOF
[Unit]
Description=dymension node service
After=network-online.target

[Service]
User=$USER
ExecStart=$(which cosmovisor) run start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
Environment="DAEMON_HOME=$HOME/.dymension"
Environment="DAEMON_NAME=dymd"
Environment="UNSAFE_SKIP_BACKUP=true"
Environment="PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:$HOME/.dymension/cosmovisor/current/bin"

[Install]
WantedBy=multi-user.target
EOF
```
```
sudo systemctl daemon-reload
sudo systemctl enable dymd.service
```
### Ayalamalar
```
dymd config node tcp://localhost:${DYMENSION_PORT}57
dymd config chain-id dymension_1100-1
dymd config keyring-backend file
```
### İnit
```
dymd init $MONIKER --chain-id dymension_1100-1
```
### Genesis addrbook
```
curl -Ls http://37.120.189.81/dymension_mainnet/genesis.json > $HOME/.dymension/config/genesis.json
curl -Ls http://37.120.189.81/dymension_mainnet/addrbook.json > $HOME/.dymension/config/addrbook.json
```
### Port
```
sed -i -e "s%:1317%:20517%; s%:8080%:20580%; s%:9090%:20590%; s%:9091%:20591%; s%:8545%:20545%; s%:8546%:20546%; s%:6065%:20565%" $HOME/.dymension/config/app.toml
sed -i -e "s%:26658%:20558%; s%:26657%:20557%; s%:6060%:20560%; s%:26656%:20556%; s%:26660%:20561%" $HOME/.dymension/config/config.toml

```
### Puring
```
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "19"|' \
  $HOME/.dymension/config/app.toml
```
### Gas prometheus indexer
```
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"20000000000adym\"|" $HOME/.dymension/config/app.toml
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.dymension/config/config.toml
sed -i -e "s/^indexer *=.*/indexer = \"null\"/" $HOME/.dymension/config/config.toml
```
### Seed And Peer
```
sed -i -e 's|^seeds *=.*|seeds = "45bffa41836302b06310af67f012500cc0d1da31@rpc.dymension.nodestake.org:666,ebc272824924ea1a27ea3183dd0b9ba713494f83@dymension-mainnet-seed.autostake.com:27086,20e1000e88125698264454a884812746c2eb4807@seeds.lavenderfive.com:20556,400f3d9e30b69e78a7fb891f60d76fa3c73f0ecc@dymension.rpc.kjnodes.com:14659,193262e32a9d7d3fffe14073160cabc4cdfef26b@dymension-rpc.stakeandrelax.net:20556,8542cd7e6bf9d260fef543bc49e59be5a3fa9074@seed.publicnode.com:26656,c28827cb96c14c905b127b92065a3fb4cd77d7f6@seeds.whispernode.com:20556,10ed1e176d874c8bb3c7c065685d2da6a4b86475@seed-dymension.ibs.team:16676,86bd5cb6e762f673f1706e5889e039d5406b4b90@seed.dymension.node75.org:10956,258f523c96efde50d5fe0a9faeea8a3e83be22ca@seed.mainnet.dymension.aviaone.com:10290"|' $HOME/.dymension/config/config.toml
```
### Snap
```
curl -L http://37.120.189.81/dymension_mainnet/dym_snap.tar.lz4 | tar -Ilz4 -xf - -C $HOME/.dymension
[[ -f $HOME/.dymension/data/upgrade-info.json ]] && cp $HOME/.dymension/data/upgrade-info.json $HOME/.dymension/cosmovisor/genesis/upgrade-info.json
```
### Başlatalım
```
sudo systemctl daemon-reload
sudo systemctl enable dymd
sudo systemctl restart dymd && sudo journalctl -u dymd -fo cat
```

### cüzdan olusturuyoruz yada import ediyoruz
```
dymd keys add cüzdan-adı 
```
```
dymd keys add cüzdan-adı --recover
```
### Create Validator
```
dymd tx staking create-validator \
--amount 1000000000000000000adym \
--pubkey $(dymd tendermint show-validator) \
--moniker "YOUR_MONIKER_NAME" \
--identity "YOUR_KEYBASE_ID" \
--details "YOUR_DETAILS" \
--website "YOUR_WEBSITE_URL" \
--chain-id dymension_1100-1 \
--commission-rate 0.05 \
--commission-max-rate 0.20 \
--commission-max-change-rate 0.05 \
--min-self-delegation 1 \
--from wallet \
--gas-adjustment 1.4 \
--gas auto \
--gas-prices 20000000000adym \
-y
  ```
## Silme Kodu
 ```
sudo systemctl stop dymd
sudo systemctl disable dymd
sudo rm -rf /etc/systemd/system/dymd.service
sudo rm $(which dymension)
sudo rm -rf $HOME/.dymension
sudo rm -rf $HOME/dymension
sed -i "/DYMENSİON_/d" $HOME/.bash_profile
 ```
