# D3 Solana Contracts

D3 Protocol is a decentralized domain name registration and management system built on Solana blockchain. These Solana contracts follows the specification of D3 EVM contracts.

## Prerequisites

Before you begin, ensure you have the following installed:

### 1. Rust and Cargo
```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source $HOME/.cargo/env
rustup component add rustfmt clippy
```

### 2. Solana Tool Suite
```bash
sh -c "$(curl -sSfL https://release.solana.com/v1.18.17/install)"

# Add to PATH (add to your .bashrc or .zshrc)
export PATH="$HOME/.local/share/solana/install/active_release/bin:$PATH"

# Verify installation
solana --version
```

### 3. Anchor Framework
```bash
# Install Anchor CLI
cargo install --git https://github.com/coral-xyz/anchor avm --locked --force
avm install latest
avm use latest

# Verify installation
anchor --version
```

### 4. Node.js and Yarn
```bash
# Install specific Node.js version
nvm install 21.6.2
nvm use 21.6.2

# Verify Node.js version
node --version  # Should output v21.6.2

```

## Setting Up the Project

### 1. Clone the Repository
```bash
git clone https://github.com/your-org/d3-protocol.git
cd d3-protocol
```

### 2. Install Dependencies
```bash
npm install
```

### 3. Configure Solana Keypairs

#### Create Development Keypair
```bash
# Create new keypair
solana-keygen new -o ~/.config/solana/id.json

# Check keypair path
solana config get

# Fund your account on devnet
solana airdrop 2 <YOUR_PUBLIC_KEY> --url devnet # Or use "https://faucet.solana.com/"
```

#### Configure Anchor.toml
```toml
[toolchain]

[features]
resolution = true
skip-lint = false

[programs.localnet]
d_3 = "Your_Program_ID"

[registry]
url = "https://api.apr.dev"

[provider]
cluster = "Devnet"
wallet = "~/.config/solana/id.json"  # Path to your keypair

[scripts]
test = "yarn run ts-mocha -p ./tsconfig.json -t 1000000 tests/**/*.ts"
```

### 4. Build the Program
```bash
# Build the program
anchor build

# Get Program ID
solana address -k target/deploy/d3_protocol-keypair.json

# Sync your program keys
anchor keys sync

# Build again 
anchor build 

# deploy program
anchor deploy
```

### 5. Running Tests
```bash
# Run tests
anchor test 

# Run tests if deployment is already done
anchor test --skip-deploy
```

### Common Issues and Solutions

1. Keypair Not Found
```bash
Error: Unable to find keypair file
Solution: Ensure keypair path in Anchor.toml matches your actual keypair location
```

2. Insufficient Funds
```bash
Error: Insufficient funds
Solution: Run solana airdrop 2 <PUBLIC_KEY> --url devnet
```

3. Program ID Mismatch
```bash
Error: Program ID mismatch
Solution: Update Program ID in lib.rs and Anchor.toml after building
```
