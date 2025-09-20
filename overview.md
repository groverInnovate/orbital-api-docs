# Overview

## What is Orbital Pool API?

The Orbital Pool API is a RESTful interface for interacting with the Orbital Pool protocol - a revolutionary 5-token Automated Market Maker (AMM) that uses torus-based invariant mathematics for superior price discovery and liquidity management.

## Key Concepts

### 5-Token AMM Architecture

Unlike traditional 2-token AMMs, Orbital Pool manages liquidity across 5 tokens simultaneously:

- **MUSDC-A** (Token Index: 0)
- **MUSDC-B** (Token Index: 1) 
- **MUSDC-C** (Token Index: 2)
- **MUSDC-D** (Token Index: 3)
- **MUSDC-E** (Token Index: 4)

### Torus-Based Invariant

The protocol uses advanced mathematical models based on torus geometry to:
- **Optimize price discovery** across multiple token pairs
- **Minimize impermanent loss** for liquidity providers
- **Enable efficient multi-hop swaps** within a single pool

### Tick-Based Liquidity Management

Orbital Pool implements a sophisticated tick system:

- **Interior Ticks** - Liquidity within mathematical constraints
- **Boundary Ticks** - Liquidity at constraint boundaries
- **Radius Calculations** - Dynamic liquidity distribution

## API Design Philosophy

### Secure by Design

- **No Private Keys** - API never handles or stores private keys
- **Unsigned Transactions** - Returns transaction data for client-side signing
- **Stateless** - No user session management required

### Developer-First

- **Simple REST** - Standard HTTP methods and JSON responses
- **Language Agnostic** - Works with any programming language
- **Comprehensive Errors** - Detailed error messages and codes

### Production Ready

- **Gas Optimization** - Built-in gas estimation and optimization
- **Rate Limiting** - Configurable request limits
- **Monitoring** - Health checks and status endpoints

## Network Information

### Arbitrum Sepolia Testnet

- **Chain ID**: 421614
- **RPC URL**: `https://sepolia-rollup.arbitrum.io/rpc`
- **Pool Contract**: `0x83EC719A6F504583d0F88CEd111cB8e8c0956431`
- **Block Explorer**: [Arbiscan Sepolia](https://sepolia.arbiscan.io)

### Supported Operations

| Operation | Description | Endpoint |
|-----------|-------------|----------|
| **Token Swaps** | Exchange one token for another | `POST /swap` |
| **Add Liquidity** | Provide liquidity to earn fees | `POST /liquidity/add` |
| **Remove Liquidity** | Withdraw liquidity and fees | `POST /liquidity/remove` |
| **Pool Status** | Get pool health and info | `GET /health` |
| **Token Info** | Get supported tokens | `GET /tokens` |
| **Gas Price** | Current network gas price | `GET /gas-price` |

## Use Cases

### For DeFi Applications

- **DEX Aggregators** - Include Orbital Pool in routing algorithms
- **Portfolio Managers** - Rebalance portfolios using 5-token swaps
- **Yield Farming** - Provide liquidity to earn trading fees

### For Traders

- **Arbitrage Bots** - Exploit price differences across pools
- **Market Makers** - Provide liquidity and earn fees
- **Multi-Token Strategies** - Execute complex trading strategies

### For Developers

- **Wallet Integration** - Add swap functionality to wallets
- **DeFi Protocols** - Integrate as a liquidity source
- **Analytics Platforms** - Track pool performance and metrics

## Getting Started

Ready to integrate? Check out our [Getting Started Guide](./getting-started.md) for step-by-step instructions.

## Architecture Diagram

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Your App     │    │  Orbital API    │    │  Orbital Pool   │
│                │    │                 │    │   Contract      │
├─────────────────┤    ├─────────────────┤    ├─────────────────┤
│ • Frontend      │───▶│ • REST Endpoints│───▶│ • 5-Token AMM   │
│ • Backend       │    │ • Gas Estimation│    │ • Torus Math    │
│ • Mobile App    │    │ • Error Handling│    │ • Tick System   │
│ • Trading Bot   │    │ • Rate Limiting │    │ • Fee Distribution│
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
         │                       │                       │
         ▼                       ▼                       ▼
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   MetaMask      │    │   Monitoring    │    │  Arbitrum       │
│   Wallet        │    │   & Analytics   │    │  Sepolia        │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

---

**Next**: [Getting Started →](./getting-started.md)
