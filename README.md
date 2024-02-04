# Namada

## NODE SETUP

### Install Essentials

```
sudo apt update; sudo apt upgrade -y
sudo apt install make clang pkg-config libssl-dev build-essential git jq llvm libudev-dev -y
```

### Install Rust

```
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y 
source $HOME/.cargo/env
```

### Install Protocol Buffers

```
sudo apt update
sudo apt install protobuf-compiler
```

### Set Variables

```
CHAIN_ID="shielded-expedition.b40d8e9055"
KEY_NAME="<YOUR KEY NAME>"
VAL_NAME="<YOUR VALIDATOR NAME>"
EMAIL="<YOUR EMAIL>"
```

### Make Variables Permanent

```
echo 'export CHAIN_ID='\"${CHAIN_ID}\" >> $HOME/.bash_profile
echo 'export KEY_NAME='\"${KEY_NAME}\" >> $HOME/.bash_profile
echo 'export VAL_NAME='\"${VAL_NAME}\" >> $HOME/.bash_profile
echo 'export EMAIL='\"${EMAIL}\" >> $HOME/.bash_profile
```

### Check the variables

```
cat ~/.bash_profile
```

### Install pre-compiled Namada Binaries - v0.31.0

```
wget https://github.com/anoma/namada/releases/download/v0.31.0/namada-v0.31.0-Linux-x86_64.tar.gz
tar xf namada-*.tar.gz
sudo mv namada-v0.31.0-Linux-x86_64/namada* /usr/local/bin/
rm -rf namada*
```

### or Compile Binaries yourself 

```
git clone https://github.com/anoma/namada.git
cd namada
git checkout v0.31.0
make install
sudo chmod +x ~/.cargo/bin/namada*
sudo mv ~/.cargo/bin/namada* /usr/local/bin
 ```

### Install CometBft

```
mkdir cometbft
wget https://github.com/cometbft/cometbft/releases/download/v0.37.2/cometbft_0.37.2_linux_amd64.tar.gz
tar xvf cometbft_0.37.2_linux_amd64.tar.gz -C ./cometbft
chmod +x cometbft/cometbft
sudo mv cometbft/cometbft /usr/local/bin/
rm -rf cometbft*
 ```

### Check Version

```
namada -V
```

### Initialize Chain & Working directory

```
namada client utils join-network --chain-id $CHAIN_ID
```

### Add service file

```
sudo tee /etc/systemd/system/namadad.service << EOF
[Unit]
Description=Namada Node
After=network.target
[Service]
User=$USER
WorkingDirectory=$HOME/.local/share/namada
Type=simple
ExecStart=/usr/local/bin/namada --base-dir=$HOME/.local/share/namada node ledger run
Environment=NAMADA_CMT_STDOUT=true
Environment=TM_LOG_LEVEL=p2p:none,pex:error
RemainAfterExit=no
Restart=on-failure
RestartSec=10s
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
```

### Enable Namada service

```
sudo systemctl enable namadad.service
sudo systemctl daemon-reload
sudo systemctl restart namadad.service
```

### Check logs

```
sudo journalctl -u namadad.service -fn 50 -o cat
```

### Full node status (if u don`t use standard port 26657, change to a right one)

```
curl localhost:26657/status
```

### Sync status (false =  node synced)

```
curl -s localhost:26657/status | jq .result.sync_info.catching_up
```


# SETUP VALIDATOR

(Node must by synced)

### Generate New Key

```
namadaw gen --alias "${KEY_NAME}"
```

### or Recover key from mnemonic

```
namadaw derive --alias "${KEY_NAME}"
```

### Check Key address

```
namadaw find --alias "${KEY_NAME}"
```

### List all the keys

```
namadaw  list
```

### Check balance

```
namadac balance --owner "${KEY_NAME}"
```

### Initialize validator

```
namadac  init-validator \
 --alias "${VAL_NAME}" \
 --account-keys "${KEY_NAME}" \
 --signing-keys "${KEY_NAME}" \
 --commission-rate 0.1 \
 --max-commission-rate-change 0.1 \
 --email "${EMAIL}"
```

### Bond tokens to your validator

```
namadac  bond \
 --source "${KEY_NAME}" \
 --validator "${VAL_NAME}" \
 --amount 1000
```

# USEFUL TIPS 

### Query TX

```
namadac tx-result --tx-hash <TX HASH> --node tcp://149.50.96.147:26657
```

### Find Validator key

```
namadac find-validator --tm-address=$(curl -s localhost:26657/status | jq -r .result.validator_info.address)
```

### Send Funds

```
namada  transfer --target <RECEIVER-KEY(tnam1..)> --source <SENDER-KEY(NAME)> --amount 100 --token NAM 
```

### Check Validator Consensus Status

```
namadac validator-state --validator "<Validator address tham1...>"
```

### UNJAIL validator

```
namadac unjail-validator --validator  "<Validator address tham1...>"
```

### Claim rewards

```
namadac claim-rewards --validator "<Validator address tnam1...>"
```

### Addr-book

```
workd="$HOME/.local/share/namada/shielded-expedition.b40d8e9055/"
wget -qO ${workd}/cometbft/config/addrbook.json https://snapshots.theamsolutions.info/shielded-addrbook.json
```

# Good Luck
