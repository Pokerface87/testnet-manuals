## atlantic-1
  ​
```bash
cd $HOME
sudo apt update
sudo apt install make clang pkg-config libssl-dev build-essential git jq ncdu bsdmainutils htop -y < "/dev/null"

# Go

cd $HOME
wget -O go1.18.1.linux-amd64.tar.gz https://golang.org/dl/go1.18.1.linux-amd64.tar.gz
sudo rm -rf /usr/local/go && sudo tar -C /usr/local -xzf go1.18.1.linux-amd64.tar.gz && rm go1.18.1.linux-amd64.tar.gz
echo 'export GOROOT=/usr/local/go' >> $HOME/.bash_profile
echo 'export GOPATH=$HOME/go' >> $HOME/.bash_profile
echo 'export GO111MODULE=on' >> $HOME/.bash_profile
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bash_profile && . $HOME/.bash_profile
go version



rm -rf $HOME/sei-chain
git clone https://github.com/sei-protocol/sei-chain
cd sei-chain
git checkout caa4c602c8f4cec37b113064e0d5e9cdb7173bed
make install
sudo mv $HOME/go/bin/seid /usr/local/bin/
cd



seid init $Seid_NODENAME --chain-id atlantic-1
wget -O $HOME/.sei/config/genesis.json "https://raw.githubusercontent.com/sei-protocol/testnet/master/sei-incentivized-testnet/genesis.json"


peers="e3b5da4caea7370cd85d7738eedaec8f56c5be28@144.76.224.246:36656,a37d65086e78865929ccb7388146fb93664223f7@18.144.13.149:26656,8ff4bd654d7b892f33af5a30ada7d8239d6f467b@91.223.3.190:51656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.sei/config/config.toml


pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.sei/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.sei/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.sei/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.sei/config/app.toml



echo "[Unit]
Description=Seid Node
After=network.target
​
[Service]
User=$USER
Type=simple
ExecStart=$(which seid) start
Restart=on-failure
LimitNOFILE=65535
​
[Install]
WantedBy=multi-user.target" > $HOME/seid.service
sudo mv $HOME/seid.service /etc/systemd/system
sudo tee <<EOF >/dev/null /etc/systemd/journald.conf
Storage=persistent
EOF
sudo systemctl restart systemd-journald
sudo systemctl daemon-reload
sudo systemctl enable seid



sudo systemctl stop seid
cp $HOME/.sei/data/priv_validator_state.json $HOME/.sei
seid tendermint unsafe-reset-all --keep-addr-book --home $HOME/.sei
SNAPSHOT_FILE=$(curl -Ls https://snapshots.brocha.in/sei/atlantic-1.json | jq -r .file)
curl -L https://snapshots.brocha.in/sei/$SNAPSHOT_FILE | lz4 -dc - | tar -xf - -C $HOME/.sei
rm $HOME/.sei/data/priv_validator_state.json
cp $HOME/.sei/priv_validator_state.json $HOME/.sei/data
rm $HOME/.sei/priv_validator_state.json


sudo systemctl restart seid
sudo journalctl -u seid -f -o cat
```
