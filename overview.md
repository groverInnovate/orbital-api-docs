# Overview

## What is Orbital Pool API?

A simple REST API for interacting with the Orbital Pool - a 5-token AMM deployed on Arbitrum Sepolia.

## The Pool

**Contract Address**: `0x83EC719A6F504583d0F88CEd111cB8e8c0956431`

The Orbital Pool is an experimental AMM that manages 5 tokens:
- **MUSDC-A** (Index: 0) - `0x9666526dcF585863f9ef52D76718d810EE77FB8D`
- **MUSDC-B** (Index: 1) - `0x1921d350666BA0Cf9309D4DA6a033EE0f0a70bEC`
- **MUSDC-C** (Index: 2) - Mock token for testing
- **MUSDC-D** (Index: 3) - Mock token for testing  
- **MUSDC-E** (Index: 4) - Mock token for testing

## How It Works

### Swaps
- Exchange any of the 5 tokens for any other
- Uses torus-based mathematical invariant for pricing
- Fallback to constant product formula when needed

### Liquidity
- Add liquidity to specific "ticks" (k-values)
- Remove liquidity and claim fees
- Tick system manages liquidity distribution

### API Design
- **Stateless** - No user accounts or sessions
- **Secure** - Never handles private keys
- **Simple** - Returns unsigned transaction data for wallet signing

## What You Can Do

| Endpoint | Purpose |
|----------|---------|
| `GET /health` | Check if API and pool are working |
| `GET /tokens` | Get list of supported tokens |
| `POST /swap` | Get transaction data for token swap |
| `POST /liquidity/add` | Get transaction data to add liquidity |
| `POST /liquidity/remove` | Get transaction data to remove liquidity |
| `GET /gas-price` | Get current gas price |

## Network Details

- **Blockchain**: Arbitrum Sepolia Testnet
- **Chain ID**: 421614
- **RPC**: `https://sepolia-rollup.arbitrum.io/rpc`
- **Explorer**: [sepolia.arbiscan.io](https://sepolia.arbiscan.io)

## Requirements

To use this API you need:
- MetaMask or other Ethereum wallet
- Some Arbitrum Sepolia ETH for gas
- Test tokens (MUSDC-A, MUSDC-B, etc.)

---

**Next**: [Getting Started →](./getting-started.md)
