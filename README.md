
# Go Liberty

Based on the `go-ethereum` v1.11.6 source code with custom modifications to the mining algorithm for research and testing purposes.

## Custom Mining Algorithm

Our code has been modified to optimize the mining process and test dynamic block reward distribution. The mining algorithm uses a combined `Blake3/Keccak256` hashing function, and the block reward automatically decreases with time

### Mining Process Overview

```
+-------------------------------------------------------------+
|                        Mining Process                       |
+-------------------------------------------------------------+
| 1. Retrieve Block Header                                    |
|    - Extract header data from the block                     |
+-------------------------------------------------------------+
| 2. Calculate Target                                         |
|    - target = 2^256 / header.Difficulty                     |
+-------------------------------------------------------------+
| 3. Initialize Variables                                     |
|    - nonce = seed, powBuffer = big.Int, iterCount = 3000    |
+-------------------------------------------------------------+
| 4. Start Nonce Search                                       |
|    - Loop until a valid nonce is found or signal received   |
+-------------------------------------------------------------+
| 5. Generate SealHash                                        |
|    - Exclude Nonce/MixDigest, Keccak256 remaining fields    |
+-------------------------------------------------------------+
| 6. Concatenate SealHash and Nonce                           |
+-------------------------------------------------------------+
| 7. Perform Multiple Hashing                                 |
|    - hashResult = Blake3(sealHash + nonce)                  |
|    - Loop iterCount: hashResult = Blake3(hashResult)        |
+-------------------------------------------------------------+
| 8. Convert and Compare Result                               |
|    - Convert hashResult to big.Int (powBuffer)              |
|    - Compare powBuffer with target                          |
+-------------------------------------------------------------+
| 9. Check Result                                             |
|    - If powBuffer <= target: Found valid nonce             |
|    - Otherwise: Increment nonce and repeat                  |
+-------------------------------------------------------------+
| 10. Complete Block                                          |
|    - Set header.Nonce and header.MixDigest                  |
|    - Log and submit the block                               |
+-------------------------------------------------------------+
```
More info in discord
https://discord.gg/PpyzRqJn

## How to Deploy the Node and Start mining

### 1. Wallet Configuration

We recommend using MetaMask or any other EVM-compatible wallet.

#### Network Settings
- **Network Name:** Liberty Project testnet
- **RPC URL:** `https://rpc.libertyproject.space`
- **Chain ID:** 21102
- **Currency Symbol:** tLBRT
- **Block Explorer URL:** `https://explorer.libertyproject.space`

Simply input these settings in your wallet to connect seamlessly with the Liberty network.

### 2. Create User


```bash
useradd -m liberty
```

### 3. Create Directories to Store Blockchain Data and Node Software

```bash
mkdir -p ~/liberty
```

### 4. Download Node Software

```bash
curl -L https://github.com/LibertyProject-chain/Liberty-Project-testnet-phase-2/releases/download/v0.23/geth-linux-amd64 -o /bin/geth
chmod +x /bin/geth
chown -R liberty:liberty ~/bin/geth
```

### 5. Create a Systemd Unit File

Create the file `/etc/systemd/system/liberty-node.service` with the following content:

```bash
sudo nano /etc/systemd/system/liberty-node.service
```

```ini
[Unit]
Description=Liberty Node
After=network.target

[Service]
User=liberty
Restart=on-failure
RestartSec=10
ExecStart=/bin/geth --datadir ~/liberty \
--networkid=21102 \
--port 40404 \
--http.api debug,web3,eth,txpool,net \
--mine \
--miner.threads=1 
--miner.etherbase=<miner address> \
--gcmode=archive \
--bootnodes "enode://8b7cf2ff6d30e7fe7f1b8bfd67193844504144cb002f1a369326d8cd16227f2c2a0a73ee0e658dac2663b92e1af7e2fbae8a46388dfd5db602f704ee56ca8d57@94.142.138.78:40404"

Restart=always
RestartSec=10
LimitNOFILE=4096

[Install]
WantedBy=default.target
```

**Note:** Replace `<miner address>` with your miner's Ethereum address.

### 6. Download and initialize the genesis file

```bash
curl -L https://github.com/LibertyProject-chain/Liberty-Project-testnet-phase-2/releases/download/v0.23/genesis.json -o ~/liberty/genesis.json
geth --datadir ~/liberty init ~/liberty/genesis.json
```



### 7. Enable and Start the Service

```bash
systemctl daemon-reload
systemctl enable liberty-node
systemctl start liberty-node
```

### 8. Check Status and Logs

To check the status of the node:

```bash
systemctl status liberty-node
```

To view the logs:

```bash
journalctl -u liberty-node -f --no-hostname -o cat
```

## Command to Run Geth Manually

### Full mode (mining + RPC)
### This command launches the node with mining enabled and an open RPC interface.
### Note: Replace <miner address> and <bootnode>  with your actual miner wallet address and bootnodes
```bash
geth --datadir ~/liberty --networkid=21102 --port 40404 \
--http.api debug,web3,eth,txpool,net --mine --miner.threads=1 \
--miner.etherbase=<miner address> --http --http.addr "0.0.0.0" \
--http.port 9945 --http.corsdomain '*' --gcmode=archive \
--bootnodes <bootnode> \
--bootnodes <bootnode2>
```

### RPC only mode (no mining)
### This command launches the node with only the RPC interface enabled, without mining.
```bash
geth --datadir ~/liberty --networkid=21102 --port 40404 \
--http.api debug,web3,eth,txpool,net --http --http.addr "0.0.0.0" \
--http.port 9945 --http.corsdomain '*' --gcmode=archive \
--bootnodes <bootnode> \
--bootnodes <bootnode2>
```

### Node only mode (no RPC and no mining)
### This command launches the node in node-only mode with syncing and connectivity.
```bash
geth --datadir ~/liberty --networkid=21102 --port 40404 \
--gcmode=archive \
--bootnodes <bootnode> \
--bootnodes <bootnode2>
```


### Command Line Options

| Option              | Description                                                                                       |
|---------------------|---------------------------------------------------------------------------------------------------|
| `--datadir`        | Specifies the data directory for the blockchain data.                                             |
| `--networkid`      | Defines the unique ID for the private network.                                                    |
| `--port`           | Specifies the network listening port for peer-to-peer communication.                              |
| `--http.api`       | Defines the APIs exposed over the HTTP RPC interface.                                             |
| `--mine`           | Enables mining on this node.                                                                      |
| `--miner.threads`  | Sets the number of CPU threads used for mining.                                                   |
| `--miner.etherbase`| Specifies the miner’s wallet address to receive mining rewards.                                   |
| `--http`           | Enables the HTTP-RPC server.                                                                      |
| `--http.addr`      | Sets the HTTP-RPC server address.                                                                 |
| `--http.port`      | Defines the HTTP-RPC server port.                                                                 |
| `--http.corsdomain`| Specifies allowed domains for CORS requests.                                                      |
| `--gcmode=archive` | Enables archival mode for retaining the full chain state.                                         |
| `--bootnodes`      | Adds boot nodes for initial connections to the network.                                           |

These options can be used to customize the node's operation and networking setup for specific requirements.

## License

This project is licensed under the MIT License.
