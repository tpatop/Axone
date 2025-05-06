[Russian version](README.ru.md)

---

# Axone

Axone Protocol is a decentralized orchestration protocol designed for efficient exchange of digital resources within the AI ecosystem. The protocol ensures fair rewards for all participants — from data providers to model developers — preventing value concentration in the hands of large corporations.

## Key Features

* Collaborative orchestration of AI workflows
* Compatible with any data, models, and infrastructure
* Flexible access rights and exchange process settings
* Custom environments with configurable rules and tokens
* Supports both on-chain and off-chain resource exchanges

## Network

| Type    | Network          |
| ------- | ---------------- |
| Testnet | axone-dentrite-1 |

## Useful Links

* [Website](https://www.axone.xyz/)
* [Documentation](https://docs.axone.xyz/)
* [Explorer](https://explore.axone.xyz/Axone%20testnet)
* [Discord](https://discord.gg/YnNedNEwxB)
* [Twitter](https://x.com/axonexyz)

---

## Node Installation Guide

### 0. Create a separate user for the node (recommended)

For better security, it is recommended to run the node under a separate user, e.g., `axone`.

#### Create the user

```bash
sudo useradd -m -s /bin/bash axone
```

#### (Optional) Add sudo privileges

```bash
sudo usermod -aG sudo axone
```

#### Switch to `axone` user

```bash
sudo su - axone
```

#### Verify the user

```bash
whoami
```

From now on, all steps are done under the `axone` user.

---

### 1. Install Go and Cosmovisor

> 💡 **Note:** If you already have Go and Cosmovisor installed, skip this step!

#### Install Go (v1.23.4)

```bash
sudo rm -rvf /usr/local/go/
wget https://golang.org/dl/go1.23.4.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.23.4.linux-amd64.tar.gz
rm go1.23.4.linux-amd64.tar.gz
```

#### Set Go environment variables

Add to `~/.profile`:

```bash
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export GO111MODULE=on
export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin
```

Apply the changes:

```bash
source ~/.profile
```

#### Install Cosmovisor (v1.0.0)

```bash
go install github.com/cosmos/cosmos-sdk/cosmovisor/cmd/cosmovisor@v1.0.0
```

---

### 2. Install the node

#### Clone and build node binary

```bash
git clone https://github.com/axone-protocol/axoned axone
cd axone
git checkout v10.0.0
make install
```

---

### 3. Node setup

#### Initialize node

Replace `YOUR_MONIKER` with your unique moniker:

```bash
axoned init YOUR_MONIKER --chain-id axone-dentrite-1
```

#### Download genesis file

```bash
wget -O genesis.json https://raw.githubusercontent.com/axone-protocol/networks/911b2d34631ac242e9ef3be577163653ed644726/chains/dentrite-1/genesis.json --inet4-only
mv genesis.json ~/.axoned/config
```

#### Setup seeds

```bash
sed -i 's/seeds = ""/seeds = "ade4d8bc8cbe014af6ebdf3cb7b1e9ad36f412c0@testnet-seeds.polkachu.com:17656"/' ~/.axoned/config/config.toml
```

If you face port conflicts, adjust ports in `~/.axoned/config/config.toml`.

---

### 4. Start the node

#### Prepare Cosmovisor folders

```bash
mkdir -p ~/.axoned/cosmovisor/genesis/bin
mkdir -p ~/.axoned/cosmovisor/upgrades

cp ~/go/bin/axoned ~/.axoned/cosmovisor/genesis/bin
```

#### Create systemd service

Create the file `/etc/systemd/system/axone.service`:

```bash
sudo nano /etc/systemd/system/axone.service
```

Paste this content:

```ini
[Unit]
Description=Axone Node
After=network-online.target

[Service]
User=axone
ExecStart=/home/axone/go/bin/cosmovisor start
Restart=always
RestartSec=3
LimitNOFILE=4096
Environment="DAEMON_NAME=axoned"
Environment="DAEMON_HOME=/home/axone/.axoned"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=false"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="UNSAFE_SKIP_BACKUP=true"

[Install]
WantedBy=multi-user.target
```

---

### 5. Load snapshot

Use snapshot from [p10node](https://docs.p10node.com/services/axone-testnet/install.html):

```bash
# Backup priv_validator_state.json
cp $HOME/.axoned/data/priv_validator_state.json $HOME/.axoned/priv_validator_state.json.backup

# Reset data
axoned tendermint unsafe-reset-all --home $HOME/.axoned --keep-addr-book

# Download and extract snapshot
curl -L https://files.p10node.com/axone-testnet/latest.tar.lz4 | tar -Ilz4 -xf - -C $HOME/.axoned

# Restore priv_validator_state.json
mv $HOME/.axoned/priv_validator_state.json.backup $HOME/.axoned/data/priv_validator_state.json
```

---

### 6. Start the node service

```bash
# Enable the service
sudo systemctl enable axone.service

# Start the service
sudo systemctl start axone.service

# Check logs
sudo journalctl -fu axone
```

---

### 7. Check node status

```bash
curl http://localhost:26657/status | jq
```

---

## Create Validator

Make sure your node is fully synced before proceeding.

### Step 1. Create operator account

```bash
axoned keys add operator
```

> ⚠️ **Important:** Save your mnemonic and private key securely!

### Step 2. Get account address

```bash
axoned keys show operator -a
```

### Step 3. Check balance

```bash
axoned query bank balances $(axoned keys show operator -a)
```

Ensure you have at least **1 AXON** (1,000,000 uaxon). Faucet link: [https://faucet.axone.xyz/](https://faucet.axone.xyz/)

---

### Step 4. Get validator pubkey

```bash
axoned tendermint show-validator
```

Copy the output — it will be used in `validator.json`.

---

### Step 5. Create `validator.json`

Create the file:

```bash
nano ~/.axoned/config/validator.json
```

> Save changes: CTRL+S and CTRL+X

Paste and modify the content:

```json
{
  "pubkey": {
    "@type": "/cosmos.crypto.ed25519.PubKey",
    "key": "<PASTE_YOUR_PUBKEY_FROM_STEP_4>"
  },
  "amount": "1000000uaxon",
  "moniker": "<YOUR_MONIKER>",
  "identity": "<YOUR_IDENTITY (optional, e.g., keybase ID)>",
  "website": "<YOUR_WEBSITE (optional)>",
  "security": "<YOUR_EMAIL (optional)>",
  "details": "<Validator description (optional)>",
  "commission-rate": "0.10",
  "commission-max-rate": "0.20",
  "commission-max-change-rate": "0.01",
  "min-self-delegation": "1"
}
```

> ✅ Replace all `<...>` fields with your data!

---

### Step 6. Create validator (send transaction)

```bash
axoned tx staking create-validator validator.json --from operator --chain-id axone-dentrite-1 --gas-adjustment 1.5 --gas auto -y
```

---

## Extra: Delegate more to your validator

```bash
axoned tx staking delegate $(axoned keys show operator --bech val -a) 1000000uaxon --chain-id axone-dentrite-1 --from operator --gas auto --gas-adjustment 1.5 -y
```

### Check validator

Check your validator on the [explorer](https://explore.axone.xyz/Axone%20testnet/staking)
