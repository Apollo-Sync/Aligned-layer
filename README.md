# Aligned-layer
Website: alignedlayer.com

Twitter: https://x.com/alignedlayer

Discord: https://discord.com/invite/alignedlayer


# Minimum hardware requirement

4 Cores, 16G Ram, 2160G SSD, Ubuntu 22.04

# Install
Update system
```
sudo apt update
sudo apt upgrade --y
```
# Install GO
```
ver="1.21.5"
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
go version
```
# Install node
```
cd $HOME
rm -rf $HOME/aligned_layer_tendermint && git clone --depth 1 --branch v0.0.2 https://github.com/yetanotherco/aligned_layer_tendermint && cd
$HOME/aligned_layer_tendermint/cmd/alignedlayerd && go build 
chmod +x alignedlayerd
sudo mv alignedlayerd /usr/local/bin/
cd $HOME
alignedlayerd version
```
# Initialize Node: Replace NodeName with your own moniker.
```
alignedlayerd init NodeName --chain-id alignedlayer
```

# Download Genesis & Download addrbook
```
wget https://raw.githubusercontent.com/Apollo-Sync/Aligned-layer/main/genesis.json

```

# Seed & Peer 
```
PEERS=a1a98d9caf27c3363fab07a8e57ee0927d8c7eec@128.140.3.188:26656,1beca410dba8907a61552554b242b4200788201c@91.107.239.79:26656,f9000461b5f535f0c13a543898cc7ac1cd10f945@88.99.174.203:26656,ca2f644f3f47521ff8245f7a5183e9bbb762c09d@116.203.81.174:26656
STAKETAB_PEER=dc2011a64fc5f888a3e575f84ecb680194307b56@148.251.235.130:20656
sed -i -e 's|^persistent_peers *=.*|persistent_peers = "'$STAKETAB_PEER','$PEERS'"|' $HOME/.alignedlayer/config/config.toml
sed -i -e 's|^seeds *=.*|seeds = "'$SEEDS'"|' $HOME/.alignedlayer/config/config.toml  
```
# Mini Gas
```
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0.0001stake\"|" $HOME/.alignedlayer/config/app.toml
```
# Create Service
```
sudo tee /etc/systemd/system/alignedlayerd.service > /dev/null <<EOF
[Unit]
Description=alignedlayerd
After=network-online.target
[Service]
User=root
ExecStart=$(which alignedlayerd) start
Restart=always
RestartSec=3
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF

cd $HOME
curl -L https://snap.nodex.one/alignedlayer-testnet/alignedlayer-latest.tar.lz4 | tar -Ilz4 -xf - -C $HOME/.alignedlayer

cd $HOME
sudo systemctl daemon-reload
sudo systemctl enable alignedlayerd
```
# Launch Node
```
sudo systemctl restart alignedlayerd
sudo journalctl -u alignedlayerd -f -o cat
```

# Wallet
**Add New Wallet Key (remember Save seed)**
```
alignedlayerd keys add wallet
```

**Recover existing key**
```
alignedlayerd keys add wallet --recover
```

**List All Keys**
```
alignedlayerd keys list
```

**Query Wallet Balance**
```
alignedlayerd q bank balances $(alignedlayerd keys show wallet -a)
```

**Check sync status (False is synced)**
```
alignedlayerd status 2>&1 | jq .sync_info
```

# VALIDATOR
**Obtain your validator public key by running the following command:**
```
alignedlayerd comet show-validator
```

**The output will be similar to this (with a different key):**
{"@type":"/cosmos.crypto.ed25519.PubKey","key":"lR1d7YBVK5jYijOfWVKRFoWCsS4dg3kagT7LB9GnG8I="}

**Then, create a file named validator.json with the following content:**
```
nano $HOME/validator.json
```

**Change your info, from "pubkey" to "details" and Save them**
```
{    
    "pubkey": {"@type":"/cosmos.crypto.ed25519.PubKey","key":"lR1d7YBVK5jYijOfWVKRFoWCsS4dg3kagT7LB9GnG8I="},
    "amount": "1000000stake",
    "moniker": "your-node-moniker",
    "identity": "eqlab testnet validator",
    "website": "optional website for your validator",
    "security": "optional security contact for your validator",
    "details": "optional details for your validator",
    "commission-rate": "0.1",
    "commission-max-rate": "0.2",
    "commission-max-change-rate": "0.01",
    "min-self-delegation": "1"
}
```

**Finally, we're ready to submit the transaction to create the validator:**
```
alignedlayerd tx staking create-validator $HOME/validator.json \
--from wallet --chain-id alignedlayer \
--fees 50stake
```

**Delegate Token to your own validator**
```
alignedlayerd tx staking delegate $(alignedlayerd keys show wallet --bech val -a)  1000000stake \
--from wallet --chain-id alignedlayer \
--fees 50stake
```

**Withdraw rewards and commission from your validator**
```
alignedlayerd tx distribution withdraw-rewards $(alignedlayerd keys show wallet --bech val -a) \
--from wallet --chain-id alignedlayer \
--fees 50stake
```

**Unjail validator**
```
alignedlayerd tx slashing unjail --from wallet --chain-id alignedlayer --fees=50stake -y
```

**Backup Validator**
```
cat $HOME/.alignedlayer/config/priv_validator_key.json
```
# Services Management
Reload Service
```
sudo systemctl daemon-reload
```

Enable Service
```
sudo systemctl enable alignedlayerd
```

Disable Service
```
sudo systemctl disable alignedlayerd
```

Start Service
```
sudo systemctl start alignedlayerd
```

Stop Service
```
sudo systemctl stop alignedlayerd
```

Restart Service
```
sudo systemctl restart alignedlayerd
```

Check Service Status
```
sudo systemctl status alignedlayerd
```

Check Service Logs
```
sudo journalctl -u alignedlayerd -f --no-hostname -o cat
```


