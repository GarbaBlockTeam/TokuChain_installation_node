# Guide for setting up a validator node for TokuChain
---

### **Step 1: Ensure System Requirements**
- **CPU:** 4 cores
- **RAM:** 8 GB
- **Disk:** 200 GB SSD
- **Operating System:** Ubuntu 20.04 or 22.04 LTS

---

### **Step 2: Install Dependencies**
Update your system and install the necessary tools:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install curl wget git build-essential libssl-dev pkg-config libclang-dev cmake -y
```

Install Rust, which is required to build the Substrate-based node:

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source $HOME/.cargo/env
rustup default stable
rustup update
```

---

### **Step 3: Clone and Build TokuChain**
Clone the TokuChain repository (replace the URL with the official repository when available) and build the binary:

```bash
git clone https://github.com/tokuchain/tokuchain-node.git
cd tokuchain-node
cargo build --release
```

The compiled binary will be located in the `target/release/` directory.

---

### **Step 4: Configure the Node**
Create a folder for the TokuChain data and configuration files:

```bash
mkdir -p ~/.tokuchain
```

Run the node for the first time to generate the default configuration:

```bash
./target/release/tokuchain-node --base-path ~/.tokuchain --chain testnet --name "<your_node_name>"
```

---

### **Step 5: Generate Keys for the Validator**
Stop the node and generate your session keys:

```bash
./target/release/tokuchain-node key generate-session-keys --chain testnet
```

Copy the output (it contains keys like `aura` and `gran`), as you’ll need it later to register your validator.

---

### **Step 6: Start the Node**
Run the node as a service using `systemd`:

```bash
sudo tee /etc/systemd/system/tokuchain.service > /dev/null <<EOF
[Unit]
Description=TokuChain Node
After=network.target

[Service]
User=$USER
ExecStart=/path/to/your/compiled/tokuchain-node --base-path ~/.tokuchain --chain testnet --name "<your_node_name>"
Restart=always
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

Enable and start the service:

```bash
sudo systemctl daemon-reload
sudo systemctl enable tokuchain
sudo systemctl start tokuchain
```

Check the logs to ensure the node is running:

```bash
journalctl -u tokuchain -f
```

---

### **Step 7: Bond Tokens**
To become a validator, bond your tokens:

1. Use the Polkadot.js UI to connect to the TokuChain testnet.
2. Go to **Staking** > **Account actions**.
3. Click **Bond** and specify the amount of tokens to bond and your controller account.

---

### **Step 8: Register as a Validator**
Submit your session keys using the Polkadot.js UI:

1. Go to **Developer** > **Extrinsics**.
2. Select your **Controller Account** and the `session.setKeys` function.
3. Paste your previously generated session keys and submit the transaction.

---

### **Step 9: Monitor and Maintain**
Use these commands and tools to monitor your validator:

- **Sync Status:**
  ```bash
  ./target/release/tokuchain-node --base-path ~/.tokuchain --chain testnet status
  ```

- **Logs:**
  ```bash
  journalctl -u tokuchain -f
  ```

