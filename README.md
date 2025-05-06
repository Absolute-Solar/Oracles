# Cryptocurrency Project Oracles

## Overview

This repository contains the oracle implementation for our Solana-based cryptocurrency project. Oracles serve as the bridge between our blockchain and the external world, providing reliable and verifiable real-world data to on-chain programs.

![Oracle Architecture](./docs/images/oracle-architecture.png)

## Table of Contents

- [What are Oracles?](#what-are-oracles)
- [Features](#features)
- [Architecture](#architecture)
- [Supported Data Sources](#supported-data-sources)
- [Installation](#installation)
- [Configuration](#configuration)
- [Usage](#usage)
  - [Running an Oracle Node](#running-an-oracle-node)
  - [Integrating with Solana Programs](#integrating-with-solana-programs)
  - [Creating Custom Data Sources](#creating-custom-data-sources)
- [Security](#security)
- [Governance](#governance)
- [Contributing](#contributing)
- [License](#license)
- [Contact](#contact)

## What are Oracles?

Oracles are specialized services that fetch and verify external data and make it available on-chain. In a blockchain environment, smart contracts cannot directly access external data sources. Our oracle system solves this by:

1. Fetching data from trusted external sources
2. Verifying the data through multiple providers
3. Aggregating results to ensure accuracy
4. Submitting verified data on-chain
5. Providing cryptographic proofs of data authenticity

## Features

- **Decentralized Oracle Network**: Multiple independent nodes for reliability and censorship resistance
- **Fault Tolerance**: Consensus mechanism ensures accurate data even if some oracles are compromised
- **Real-time Data Feeds**: Low-latency updates for time-sensitive applications like DeFi
- **Verifiable Data**: Cryptographic proofs for all provided data
- **Custom Data Sources**: Easily add new data providers and APIs
- **Stake-based Security**: Oracles stake tokens as collateral against malicious behavior
- **Governance Integration**: Oracle parameters and data sources controlled by DAO
- **Developer-friendly SDK**: Simple integration with Solana programs

## Architecture

Our oracle system uses a layered architecture designed specifically for Solana:

### Data Provider Layer
External APIs, data feeds, and web services that contain real-world data.

### Oracle Node Layer
Distributed nodes written in Rust and TypeScript that independently fetch, verify, and submit data on-chain.

### Aggregation Layer
On-chain Solana programs that collect multiple oracle submissions and determine the consensus value.

### Consumer Layer
Solana programs that utilize the oracle data for decision-making.

## Supported Data Sources

Our oracle network currently supports the following data types:

- **Financial Data**
  - Cryptocurrency prices
  - Stock prices
  - Exchange rates
  - Interest rates
  - Commodity prices

- **Market Data**
  - Trading volume
  - Liquidity metrics
  - Order book depth

- **Weather & Environmental**
  - Temperature
  - Precipitation
  - Air quality
  - Natural disaster data

- **Sports & Events**
  - Game scores
  - Match outcomes
  - Tournament results

- **Randomness**
  - Verifiable random numbers
  - Entropy sources

- **IoT & Connected Devices**
  - Sensor readings
  - Machine states
  - Location data

## Installation

### Prerequisites

- Rust v1.65+
- Node.js v16+
- TypeScript v4.7+
- Solana CLI v1.14+
- Access to a Solana RPC node

### Steps

```bash
# Clone the repository
git clone https://github.com/yourusername/crypto-project-oracles.git
cd crypto-project-oracles

# Install dependencies
npm install

# Install Rust dependencies
cargo build

# Build the Solana program
cd program
cargo build-bpf
cd ..

# Build the TypeScript client
npm run build

# Run tests
npm test
```

## Configuration

Configure your oracle node by creating a `.env` file based on the template:

```bash
cp .env.example .env
```

Key configuration options:

```
# Network Configuration
SOLANA_RPC_URL=https://api.mainnet-beta.solana.com
SOLANA_NETWORK=mainnet-beta

# Oracle Identity
ORACLE_KEYPAIR_PATH=./keypair.json
ORACLE_STAKE_AMOUNT=5000

# Data Sources
PRICE_API_KEY=your_api_key
WEATHER_API_KEY=your_api_key

# Performance Settings
UPDATE_INTERVAL=60000
CONFIRMATION_BLOCKS=32
```

## Usage

### Running an Oracle Node

Start your oracle node with:

```bash
npm run oracle:start
```

Monitor your oracle's activity:

```bash
npm run oracle:status
```

### Integrating with Solana Programs

#### Rust Example

```rust
use solana_program::{
    account_info::{next_account_info, AccountInfo},
    entrypoint,
    entrypoint::ProgramResult,
    msg,
    program_error::ProgramError,
    pubkey::Pubkey,
};
use std::convert::TryInto;

// Import our oracle interfaces
use crypto_project_oracle_client::{OracleAccount, OracleAccountData, PriceData};

// Program entrypoint
entrypoint!(process_instruction);

// Program logic
pub fn process_instruction(
    program_id: &Pubkey,
    accounts: &[AccountInfo],
    _instruction_data: &[u8],
) -> ProgramResult {
    let accounts_iter = &mut accounts.iter();
    
    // Get account info for accounts
    let price_oracle_ai = next_account_info(accounts_iter)?;
    
    // Deserialize the oracle account
    let oracle_data = OracleAccount::deserialize(&price_oracle_ai.data.borrow())?;
    
    // Get BTC price feed
    let btc_price = match oracle_data.get_price_feed("BTC-USD") {
        Some(price) => price.price,
        None => return Err(ProgramError::InvalidAccountData),
    };
    
    // Use the price in your program logic
    msg!("Current BTC price: {}", btc_price);
    
    if btc_price > 50000 {
        // Do something when BTC price is high
    } else {
        // Do something when BTC price is lower
    }
    
    Ok(())
}
```

#### TypeScript Client Example

```typescript
import {
  Connection,
  Keypair,
  PublicKey,
  Transaction,
  TransactionInstruction,
} from '@solana/web3.js';
import { OracleClient } from '@crypto-project/oracle-client';

async function checkPrice() {
  // Connect to the Solana network
  const connection = new Connection('https://api.mainnet-beta.solana.com', 'confirmed');
  
  // Initialize the Oracle client
  const oracleClient = new OracleClient(connection);
  
  // Get the oracle program account
  const oracleProgramId = new PublicKey('Your_Oracle_Program_ID');
  
  // Get BTC price feed account
  const btcPriceFeed = await oracleClient.getPriceFeedAccount('BTC-USD');
  
  // Get the latest price
  const priceData = await oracleClient.getPriceData(btcPriceFeed);
  
  console.log(`Current BTC price: ${priceData.price}`);
  console.log(`Last updated: ${new Date(priceData.lastUpdatedTimestamp * 1000)}`);
  console.log(`Confidence interval: ${priceData.confidence}`);
}

checkPrice();
```

### Creating Custom Data Sources

You can extend the oracle system with custom data sources:

1. Create a new data source adapter in TypeScript:

```typescript
// src/datasources/custom-api.ts
import { BaseDataSource, DataSourceConfig, DataResponse } from '../core/base-datasource';

export class CustomApiDataSource extends BaseDataSource {
  constructor(config: DataSourceConfig) {
    super('CUSTOM_API', config);
  }
  
  async fetchData(): Promise<DataResponse> {
    // Implementation for fetching from your API
    const response = await fetch(this.config.endpoint);
    const data = await response.json();
    
    return this.formatResponse(data);
  }
  
  private formatResponse(rawData: any): DataResponse {
    // Transform the API response to our standard format
    return {
      value: rawData.mainMetric,
      timestamp: Date.now(),
      source: this.name,
      metadata: {
        subValues: rawData.secondaryMetrics
      }
    };
  }
}
```

2. Register your data source in the configuration:

```typescript
// src/config/datasources.ts
import { CustomApiDataSource } from '../datasources/custom-api';

export const dataSources = [
  {
    type: 'CUSTOM_API',
    config: {
      endpoint: 'https://api.example.com/data',
      apiKey: process.env.CUSTOM_API_KEY,
      refreshInterval: 60000,
    },
    feeds: [
      {
        name: 'CUSTOM_METRIC',
        path: 'result.value',
        transform: (value) => parseInt(value, 10),
      }
    ]
  }
];
```

## Security

Our oracle system implements several security measures:

- **Multiple Independent Data Sources**: Each data point is collected from multiple sources to prevent manipulation.

- **Stake-based Security**: Oracle operators must stake tokens that can be slashed for malicious behavior.

- **Cryptographic Proofs**: All data submissions include cryptographic signatures for verification.

- **Consensus Mechanism**: A threshold of oracles must agree on a value before it's accepted.

- **Timely Updates**: Data feeds become invalid after a configurable time period.

- **Anomaly Detection**: Statistical analysis flags unusual price movements for verification.

- **Open-source Code**: All code is open-source and audited by security professionals.

## Governance

The oracle system is governed by the project's DAO through:

- **Data Source Proposals**: Adding or removing data sources
- **Oracle Node Approvals**: Whitelisting reliable oracle operators
- **Parameter Updates**: Adjusting thresholds, update frequencies, and reward structures
- **Protocol Upgrades**: Implementing new features and security improvements

## Contributing

We welcome contributions! Here's how to get started:

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add some amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

Before submitting, please:
- Run the test suite (`npm test`)
- Update documentation for any new features
- Follow our code style guidelines

## License

This project is licensed under the Apache 2.0 License - see the [LICENSE](LICENSE) file for details.

## Contact

- Website: [your-website.com](https://your-website.com)
- Email: [your-email@example.com](mailto:your-email@example.com)
- Twitter: [@yourproject](https://twitter.com/yourproject)
- Discord: [Join our community](https://discord.gg/yourproject)
- Telegram: [t.me/yourproject](https://t.me/yourproject)

---

*This oracle system is designed to work seamlessly with our main cryptocurrency project. For more information on the broader project, please refer to the [main repository](https://github.com/yourusername/your-crypto-project).*
