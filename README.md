# PumpFun Sniper Bot

A high-performance Solana sniper bot that detects and trades new PumpFun token launches in the first block using AllenHark's ultra-low latency ShredStream service.

## 🚀 Features

- **First-Block Sniping** - Detects new PumpFun token creations via ShredStream and executes trades in the same block
- **Ultra-Low Latency** - Direct shred processing for sub-millisecond detection times
- **Token 2022 Support** - Works with both standard SPL tokens and Token 2022 (SPL-22)
- **Automatic Buy/Sell** - Configurable automatic buying and selling with profit tracking
- **Smart Filtering** - Customizable filters for dev buy amounts, token metrics, and spam detection
- **Blacklist System** - Automatic blacklisting of unprofitable creators to avoid repeat losses
- **AllenHark Integration** - Optimized for AllenHark Relay and ShredStream services

## 📋 Before You Start

You will need:

| Requirement | Description |
|-------------|-------------|
| **Solana Wallet** | A funded wallet with SOL for trading. Create one with: `solana-keygen new -o wallet.json` |
| **RPC Access** | API keys from providers like Helius, Shyft, QuickNode, or Alchemy |
| **ShredStream** | Access to AllenHark ShredStream for real-time data. Sign up at [allenhark.com](https://allenhark.com) |

## 🔧 Installation

### Download Pre-built Binaries (Recommended)

```bash
# Download the latest release
wget https://github.com/AllenHark-Inc/PumpFun-Sniper/releases/latest/download/pumpfun-sniper-linux-v*.tar.gz

# Extract
tar -xzf pumpfun-sniper-linux-*.tar.gz
cd linux

# Make executable
chmod +x sniperbot sell_token add_blacklist fix_blacklist test_blacklist
```

### Build from Source

> [!NOTE]
> Access to the private source repository requires a one-time purchase of **40 SOL**.
> Contact us on [Discord](https://discord.gg/JpzS72MAKG) or visit [AllenHark.com](https://allenhark.com) for access.

```bash
# Clone the private repository (requires access)
git clone https://github.com/AllenHark-Inc/Ark-Shreds-Consumer-Rs.git
cd Ark-Shreds-Consumer-Rs

# Build
cargo build --release --target x86_64-unknown-linux-gnu
```

---

## ⚙️ Configuration Reference

Create a `config.toml` file with the following options:

### Trading Settings

| Option | Type | Description |
|--------|------|-------------|
| `purchase_amount_sol` | Number | Amount of SOL to spend per snipe (e.g., `0.001`) |
| `slippage_basis_points` | Number | Slippage tolerance in basis points. `100` = 1%, `200` = 2% |
| `live` | Boolean | `true` = real trading, `false` = dry-run mode (no actual transactions) |

**Example:**
```toml
purchase_amount_sol = 0.001
slippage_basis_points = 100
live = true
```

---

### Wallet Configuration

| Option | Type | Description |
|--------|------|-------------|
| `payer_keypair_path` | String | Path to your Solana wallet JSON file |

**Example:**
```toml
payer_keypair_path = "./wallet.json"
```

> [!WARNING]
> Never share your wallet keypair file with anyone!

---

### AllenHark Services

| Option | Type | Description |
|--------|------|-------------|
| `shredstream_uri` | String | ShredStream endpoint URL for real-time data |
| `x_token` | String | Your ShredStream authentication token |
| `relay` | String | AllenHark Relay endpoint for fast transaction submission |
| `keep_alive_url` | String | URL to ping for keeping connection alive |
| `tip_wallet` | String | Wallet to receive Jito tips |
| `tip_lamports` | Number | Tip amount in lamports (1 SOL = 1,000,000,000 lamports) |

**Example:**
```toml
shredstream_uri = "http://127.0.0.1:9090"
x_token = "ss_your_token_here"
relay = "https://relay.allenhark.com/v1/sendTx"
keep_alive_url = "https://relay.allenhark.com/ping"
tip_wallet = "harkYFxB6DuUFNwDLvA5CQ66KpfRvFgUoVypMagNcmd"
tip_lamports = 1_000_000
```

---

### RPC Configuration

| Option | Type | Description |
|--------|------|-------------|
| `rpc_pool` | Array | List of RPC endpoints to use (load balanced) |
| `default_rpc` | String | Primary RPC endpoint for transactions |
| `default_wss_rpc` | String | WebSocket RPC for streaming blockhash updates |
| `use_fast_sender` | Boolean | Enable Jito-style fast transaction sending |

**Example:**
```toml
use_fast_sender = true
default_rpc = "https://rpc.shyft.to?api_key=YOUR_KEY"
default_wss_rpc = "wss://rpc.shyft.to?api_key=YOUR_KEY"

rpc_pool = [
    "https://mainnet.helius-rpc.com/?api-key=YOUR_KEY",
    "https://rpc.shyft.to?api_key=YOUR_KEY",
    "https://solana-mainnet.g.alchemy.com/v2/YOUR_KEY"
]
```

---

### PumpFun Settings

| Option | Type | Description |
|--------|------|-------------|
| `user_volume_accumulator` | String | PDA derived from your wallet (auto-generated on first run) |

This is automatically calculated from your wallet. You don't need to set it manually.

---

### Dev Buy Filter

Filter tokens based on how much SOL the dev spent on their initial buy:

| Option | Type | Description |
|--------|------|-------------|
| `min_dev_buy_sol` | Number | Minimum SOL the dev must spend (skip cheap launches) |
| `max_dev_buy_sol` | Number | Maximum SOL the dev can spend (skip potential dumps) |

**Example:**
```toml
min_dev_buy_sol = 0.5    # Only snipe if dev bought with 0.5+ SOL
max_dev_buy_sol = 4.5    # Skip if dev bought with more than 4.5 SOL
```

**Why filter dev buys?**
- **Too low** (< 0.5 SOL): Often spam or test tokens
- **Too high** (> 5 SOL): Dev might dump immediately

---

### Transaction Timing

| Option | Type | Description |
|--------|------|-------------|
| `confirmation_wait_ms` | Number | Milliseconds to wait before selling after buy |
| `balance_check_max_retries` | Number | How many times to retry checking token balance |

**Example:**
```toml
confirmation_wait_ms = 2301      # Wait ~2.3 seconds before selling
balance_check_max_retries = 5   # Retry up to 5 times
```

---

### Spam Filter

Additional filters to avoid spam tokens. Place under `[spam_filter]` section:

| Option | Type | Description |
|--------|------|-------------|
| `min_buy_sol` | Number | Minimum buy amount to consider |
| `max_buy_sol` | Number | Maximum buy amount to consider |
| `min_tokens` | Number | Minimum token amount in transaction |
| `min_accounts` | Number | Minimum accounts in transaction (real tokens have more) |

**Example:**
```toml
[spam_filter]
min_buy_sol = 0.01
max_buy_sol = 10.0
min_tokens = 1000
min_accounts = 8
```

---

### Account Whitelist (Optional)

Only snipe tokens from specific wallet addresses:

| Option | Type | Description |
|--------|------|-------------|
| `account_include` | Array | List of wallet addresses to whitelist |

**Example:**
```toml
account_include = [
    "ABC123...",
    "DEF456..."
]
```

Leave this unset to snipe all qualifying tokens.

---

## 📄 Complete Configuration Example

```toml
# === TRADING ===
purchase_amount_sol = 0.001
slippage_basis_points = 100
live = true

# === WALLET ===
payer_keypair_path = "./wallet.json"

# === ALLENHARK SERVICES ===
shredstream_uri = "http://127.0.0.1:9090"
x_token = "ss_your_token_here"
relay = "https://relay.allenhark.com/v1/sendTx"
keep_alive_url = "https://relay.allenhark.com/ping"
tip_wallet = "harkYFxB6DuUFNwDLvA5CQ66KpfRvFgUoVypMagNcmd"
tip_lamports = 1_000_000

# === RPC ===
use_fast_sender = true
default_rpc = "https://rpc.shyft.to?api_key=YOUR_KEY"
default_wss_rpc = "wss://rpc.shyft.to?api_key=YOUR_KEY"
rpc_pool = [
    "https://mainnet.helius-rpc.com/?api-key=YOUR_KEY",
    "https://rpc.shyft.to?api_key=YOUR_KEY"
]

# === FILTERS ===
min_dev_buy_sol = 0.5
max_dev_buy_sol = 4.5
confirmation_wait_ms = 2301
balance_check_max_retries = 5

[spam_filter]
min_buy_sol = 0.01
max_buy_sol = 10.0
min_tokens = 1000
min_accounts = 8

# account_include = []  # Optional: whitelist specific creators
```

---

## 🎯 Usage

### Start the Bot

```bash
./sniperbot
```

The bot will:
1. Connect to ShredStream
2. Monitor for new PumpFun tokens (both standard and Token 2022)
3. Filter based on your settings
4. Buy qualifying tokens
5. Sell after confirmation
6. Track profits and update blacklist

### Utility Commands

| Command | Description |
|---------|-------------|
| `./sell_token --token <MINT>` | Manually sell a specific token |
| `./add_blacklist <WALLET>` | Block a creator wallet |
| `./test_blacklist <WALLET>` | Check if wallet is blacklisted |
| `./fix_blacklist` | Repair corrupted blacklist |

---

## 💰 Fees

> [!IMPORTANT]
> The bot charges **0.001 SOL before each buy** and **0.001 SOL after each sell** (total **0.002 SOL per trade cycle**).

**Remove Fees:** Purchase source code access for **40 SOL** to remove all fees. Contact us on [Discord](https://discord.gg/JpzS72MAKG).

---

## 📊 Recommended Settings

### 🟢 Beginner (Safe)
```toml
purchase_amount_sol = 0.001
slippage_basis_points = 200
min_dev_buy_sol = 1.0
max_dev_buy_sol = 3.0
live = false  # Test first!
```

### 🟡 Moderate
```toml
purchase_amount_sol = 0.005
slippage_basis_points = 150
min_dev_buy_sol = 0.5
max_dev_buy_sol = 4.0
live = true
```

### 🔴 Aggressive (High Risk)
```toml
purchase_amount_sol = 0.01
slippage_basis_points = 100
min_dev_buy_sol = 0.3
max_dev_buy_sol = 5.0
confirmation_wait_ms = 1500
live = true
```

---

## ⚠️ Risk Warning

**Trading cryptocurrencies involves significant risk of loss.**

- PumpFun tokens are highly volatile
- Many tokens are scams or rug pulls
- You may lose all invested SOL
- Fees are charged regardless of profit/loss

**Only trade with funds you can afford to lose.**

---

## 🛠️ Troubleshooting

| Problem | Solution |
|---------|----------|
| Bot won't start | Check `config.toml` exists and is valid |
| No tokens sniped | Verify ShredStream is running, check filters |
| Transactions failing | Ensure wallet has SOL, increase slippage |
| Blacklist errors | Run `./fix_blacklist` |

---

## 📚 Support

- **Documentation:** [allenhark.com/docs/pumpfun-sniper](https://allenhark.com/docs/pumpfun-sniper)
- **Discord:** [discord.gg/JpzS72MAKG](https://discord.gg/JpzS72MAKG)
- **Website:** [allenhark.com](https://allenhark.com)

---

**Built with ⚡ by AllenHark** | **Powered by AllenHark ShredStream**
