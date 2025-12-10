# PumpFun Sniper Bot

A high-performance Solana sniper bot that detects and trades new PumpFun token launches in the first block using AllenHark's ultra-low latency ShredStream service.

## 🚀 Features

- **First-Block Sniping**: Detects new PumpFun token creations via ShredStream and executes trades in the same block
- **Ultra-Low Latency**: Direct shred processing for sub-millisecond detection times
- **Automatic Buy/Sell**: Configurable automatic buying and selling with profit tracking
- **Smart Filtering**: Customizable filters for dev buy amounts, token metrics, and spam detection
- **Blacklist System**: Automatic blacklisting of unprofitable creators to avoid repeat losses
- **AllenHark Integration**: Optimized for AllenHark Relay and ShredStream services

## 📋 Prerequisites

Before running the bot, you need:

1. **Funded Solana Wallet**: A Solana wallet with sufficient SOL for trading
   - Generate a keypair: `solana-keygen new -o wallet.json`
   - Fund it with SOL for trading + transaction fees

2. **RPC Endpoints**: Access to Solana RPC providers (Helius, Shyft, QuickNode, Alchemy, etc.)
   - Get free/paid API keys from your preferred RPC providers

3. **AllenHark Services** (Recommended for best performance):
   - ShredStream access for real-time shred data
   - AllenHark Relay for optimized transaction submission
   - Sign up at: https://allenhark.com

4. **User Volume Accumulator**: A PDA derived from your wallet for PumpFun volume tracking
   - See [Deriving User Volume Accumulator](#deriving-user-volume-accumulator) below

## 🔧 Installation

### Option 1: Download Pre-built Binaries (Recommended)

Download the latest release for Linux (x86_64):

```bash
# Download and extract the release
wget https://github.com/AllenHark-Inc/PumpFun-Sniper/sniperbot-linux-x86_64.tar.gz
tar -xzf sniperbot-linux-x86_64.tar.gz
cd sniperbot

# Make binaries executable
chmod +x sniperbot sell_token add_blacklist fix_blacklist test_blacklist
```

### Option 2: Build from Source

> [!NOTE]
> Access to the private source code repository requires a one-time purchase of **40 SOL**.
> Contact us via [Discord](https://discord.gg/JpzS72MAKG) or visit [AllenHark.com](https://allenhark.com) for access.

Once you have access to the private repository:

```bash
# Clone the private repository
git clone https://github.com/AllenHark-Inc/Ark-Shreds-Consumer-Rs.git
cd Ark-Shreds-Consumer-Rs

# Build for Linux
cargo build --release --target x86_64-unknown-linux-gnu

# Binaries will be in target/x86_64-unknown-linux-gnu/release/
```

## ⚙️ Configuration

### 1. Create `config.toml`

Copy the example configuration and customize it:

```toml
# Trading Parameters
purchase_amount_sol = 0.001          # Amount of SOL to spend per snipe (0.001 SOL fee per transaction)
slippage_basis_points = 100          # Slippage tolerance (100 = 1%)
live = true                          # Set to false for dry-run mode (no real transactions)

# Wallet Configuration
payer_keypair_path = "./wallet.json" # Path to your Solana wallet keypair

# AllenHark Services (for best performance)
relay = "https://relay.allenhark.com/v1/sendTx"
quic = "160.202.131.11:4433"
keep_alive_url = "https://relay.allenhark.com/ping"
shredstream_uri = "http://127.0.0.1:9999"  # Local ShredStream proxy
tip_wallet = "harkYFxB6DuUFNwDLvA5CQ66KpfRvFgUoVypMagNcmd"
tip_lamports = 1_000_000             # 0.001 SOL tip per transaction

# PumpFun Configuration
user_volume_accumulator = "YOUR_DERIVED_PDA_HERE"  # See derivation guide below

# RPC Pool (add your own RPC endpoints with API keys)
rpc_pool = [
    "https://mainnet.helius-rpc.com/?api-key=YOUR_KEY",
    "https://rpc.shyft.to?api_key=YOUR_KEY",
    "https://solana-mainnet.quiknode.pro/YOUR_KEY",
    "https://solana-mainnet.g.alchemy.com/v2/YOUR_KEY"
]

# Jito Configuration
jito_enabled = true
default_rpc = "https://rpc.shyft.to?api_key=YOUR_KEY"

# Transaction Confirmation Settings
confirmation_wait_ms = 2301          # Time to wait for transaction confirmation (ms)
balance_check_max_retries = 5        # Max retries when checking token balance

# Dev Buy Filter (skip tokens outside this range)
min_dev_buy_sol = 0.5                # Minimum dev initial buy (SOL)
max_dev_buy_sol = 4.5                # Maximum dev initial buy (SOL)

# Spam Filter
[spam_filter]
min_buy_sol = 0.01                   # Minimum buy amount to consider
max_buy_sol = 10.0                   # Maximum buy amount to consider
min_tokens = 1000                    # Minimum token amount
min_accounts = 8                     # Minimum number of accounts in transaction

# Optional: Whitelist specific accounts
# account_include = ["PUBKEY1", "PUBKEY2"]
```

### 2. Deriving User Volume Accumulator

The `user_volume_accumulator` is a PDA (Program Derived Address) required by PumpFun to track your trading volume.

#### Method 1: Using Rust

```rust
use solana_sdk::pubkey::Pubkey;
use std::str::FromStr;

fn derive_user_volume_accumulator(user_pubkey: &Pubkey) -> Pubkey {
    let pumpfun_program = Pubkey::from_str("6EF8rrecthR5Dkzon8Nwu78hRvfCKubJ14M5uBEwF6P").unwrap();
    let fee_program = Pubkey::from_str("pfeeUxB6jkeY1Hxd7CsFCAjcbHA9rWtchMGdZ6VojVZ").unwrap();
    
    let (pda, _bump) = Pubkey::find_program_address(
        &[
            b"user-volume-accumulator",
            user_pubkey.as_ref(),
            pumpfun_program.as_ref(),
        ],
        &fee_program,
    );
    
    pda
}

// Usage:
let user = Pubkey::from_str("YOUR_WALLET_PUBKEY").unwrap();
let accumulator = derive_user_volume_accumulator(&user);
println!("User Volume Accumulator: {}", accumulator);
```

#### Method 2: Using Node.js

```javascript
const { PublicKey } = require('@solana/web3.js');

function deriveUserVolumeAccumulator(userPubkey) {
    const pumpfunProgram = new PublicKey('6EF8rrecthR5Dkzon8Nwu78hRvfCKubJ14M5uBEwF6P');
    const feeProgram = new PublicKey('pfeeUxB6jkeY1Hxd7CsFCAjcbHA9rWtchMGdZ6VojVZ');
    
    const [pda, bump] = PublicKey.findProgramAddressSync(
        [
            Buffer.from('user-volume-accumulator'),
            userPubkey.toBuffer(),
            pumpfunProgram.toBuffer(),
        ],
        feeProgram
    );
    
    return pda;
}

// Usage:
const user = new PublicKey('YOUR_WALLET_PUBKEY');
const accumulator = deriveUserVolumeAccumulator(user);
console.log('User Volume Accumulator:', accumulator.toString());
```

For more details, see: https://allenhark.com/docs/pumpfun-sniper

### 3. Download Initial Blacklist (Recommended)

Download the pre-populated blacklist to avoid known malicious wallets:

```bash
# Download the latest blacklist (updated daily)
wget https://allenhark.com/blacklist.jsonl -O blacklist.jsonl

# Or use curl
curl -o blacklist.jsonl https://allenhark.com/blacklist.jsonl
```

The blacklist is updated daily with high-frequency token launchers and known rug pullers. The bot will automatically update this file as it detects unprofitable trades.

Learn more: https://allenhark.com/docs/blacklist

## 🎯 Usage

### Main Sniper Bot

Run the main sniper bot:

```bash
./sniperbot
```

The bot will:
1. Connect to ShredStream for real-time shred data
2. Monitor for new PumpFun token creations
3. Filter tokens based on your configuration
4. Automatically buy tokens that pass filters
5. Automatically sell after confirmation
6. Track profits and update blacklist

> [!IMPORTANT]
> **Usage Fees**: The bot charges **0.001 SOL before each buy** and **0.001 SOL after each sell** (total **0.002 SOL per complete trade cycle**) paid to AllenHark. These fees are 
charged regardless of trade profitability.
> 
> **Remove Fees**: Purchase source code access for **40 SOL** to remove all usage fees and customize the bot. Contact us via [Discord](https://discord.gg/JpzS72MAKG) or visit 
[AllenHark.com](https://allenhark.com).


### Utility Binaries

#### Sell Token

Manually sell a specific token if the bot skipped it or failed to sell:

```bash
./sell_token --token <MINT_ADDRESS>

# Example:
./sell_token --token GRTZHBm1X6N6rx99UowUbnTDUe5AFStJHqcw5bGApump
```

#### Add to Blacklist

Manually add a wallet address to the blacklist:

```bash
./add_blacklist <WALLET_ADDRESS>

# Example:
./add_blacklist 7xKXtg2CW87d97TXJSDpbD5jBkheTqA83TZRuJosgAsU
```

#### Test Blacklist

Test if a wallet address is in the blacklist:

```bash
./test_blacklist <WALLET_ADDRESS>

# Example:
./test_blacklist 7xKXtg2CW87d97TXJSDpbD5jBkheTqA83TZRuJosgAsU
```

#### Fix Blacklist

Fix data corruption in the blacklist file:

```bash
./fix_blacklist
```

Use this if you encounter errors loading the blacklist.

## 📊 Understanding the Output

The bot provides detailed real-time metrics:

```
⚡ Event Latency: 234 μs (direct channel, no ZMQ!)
🔄 Processing Mint: GRTZHBm1X6N6rx99UowUbnTDUe5AFStJHqcw5bGApump
🚀 PRE-FLIGHT TIME: 1.234ms (Channel -> Serialized, no ZMQ!)
✅ Token account confirmed and available
💰 Token account balance: 1234567 tokens
🔄 Preparing SELL transaction...
✅ Sell transaction sent!

💰 Profit Calculation:
  Initial SOL balance: 10.123456789 SOL
  Final SOL balance: 10.125678901 SOL
  Rent recovered: 0.002039280 SOL
  Total profit (with rent): 0.004261392 SOL
```

## 🔒 Security Best Practices

1. **Never share your wallet keypair** (`wallet.json`)
2. **Keep your RPC API keys private** - don't commit `config.toml` to public repos
3. **Start with small amounts** - test with `purchase_amount_sol = 0.001` first
4. **Use dry-run mode** - set `live = false` to test without real transactions
5. **Monitor your wallet balance** - the bot will spend SOL on every qualifying token
6. **Review the blacklist regularly** - ensure it's not blocking legitimate opportunities

## ⚠️ Risk Disclaimer

**Trading cryptocurrencies involves significant risk of loss. This bot is provided as-is with no guarantees of profit.**

- PumpFun tokens are highly volatile and risky
- Many tokens are scams or rug pulls
- You may lose all SOL spent on trades
- Past performance does not guarantee future results
- Usage fees (0.002 SOL per trade cycle) are charged regardless of trade outcome and are non-refundable
- Always trade responsibly and only with funds you can afford to lose


## 📚 Documentation & Support

- **Full Documentation**: https://allenhark.com/docs/pumpfun-sniper
- **Blacklist Guide**: https://allenhark.com/docs/blacklist
- **Discord Support**: https://discord.gg/JpzS72MAKG
- **AllenHark Services**: https://allenhark.com

## 🛠️ Troubleshooting

### Bot won't start
- Check that `config.toml` exists and is valid TOML format
- Verify your wallet keypair path is correct
- Ensure your wallet has sufficient SOL balance

### No tokens being sniped
- Verify ShredStream is running and accessible
- Check your spam filters aren't too restrictive
- Ensure your RPC endpoints are working
- Verify `live = true` in config

### Transactions failing
- Check your wallet has enough SOL for trades + fees
- Verify RPC endpoints are responsive
- Increase `slippage_basis_points` if needed
- Check AllenHark Relay is accessible

### Blacklist errors
- Run `./fix_blacklist` to repair corruption
- Delete `blacklist.jsonl` and re-download from https://allenhark.com/blacklist.jsonl
- Check file permissions

## 📝 Configuration Tips

### Recommended Settings for Beginners

```toml
purchase_amount_sol = 0.001      # Start small
slippage_basis_points = 200      # Higher slippage for volatile tokens
min_dev_buy_sol = 1.0            # Only snipe tokens with 1+ SOL dev buy
max_dev_buy_sol = 3.0            # Avoid huge dev buys (potential dumps)
live = false                     # Test in dry-run mode first
```

### Aggressive Settings (Higher Risk/Reward)

```toml
purchase_amount_sol = 0.01       # Larger position sizes
slippage_basis_points = 100      # Tighter slippage
min_dev_buy_sol = 0.5            # Cast a wider net
max_dev_buy_sol = 5.0            # Include larger dev buys
confirmation_wait_ms = 1500      # Faster selling (riskier)
```

### Conservative Settings (Lower Risk)

```toml
purchase_amount_sol = 0.001      # Minimal position size
slippage_basis_points = 300      # Accept more slippage
min_dev_buy_sol = 2.0            # Only well-funded launches
max_dev_buy_sol = 3.0            # Avoid extreme dev buys
confirmation_wait_ms = 3000      # Wait longer for confirmation
```

## 🤝 Contributing

This is an open-source project. Contributions, issues, and feature requests are welcome!

## 📄 License

[Add your license here]

---

**Built with ⚡ by AllenHark** | **Powered by ShredStream**

