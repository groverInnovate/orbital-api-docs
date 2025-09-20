# Orbital Pool API Documentation

Simple REST API for interacting with the Orbital Pool - a 5-token AMM on Arbitrum Sepolia.

## Quick Start

```bash
# Check API status
curl http://localhost:8000/health

# Execute a token swap
curl -X POST http://localhost:8000/swap \
  -H "Content-Type: application/json" \
  -d '{
    "token_in_index": 0,
    "token_out_index": 1,
    "amount_in": "1000000000000000000",
    "min_amount_out": "0",
    "user_address": "0xYourAddress"
  }'
```

## Documentation

- [**Overview**](./overview.md) - What is Orbital Pool API
- [**Getting Started**](./getting-started.md) - Basic integration
- [**API Reference**](./api-reference.md) - All endpoints
- [**Examples**](./examples.md) - Code samples
- [**Error Handling**](./error-handling.md) - Common issues

## What This API Does

- **Token Swaps** - Exchange between 5 supported tokens
- **Add Liquidity** - Provide liquidity to earn fees
- **Remove Liquidity** - Withdraw your liquidity
- **Gas Estimation** - Get current gas prices
- **Pool Status** - Check API and pool health

## Network Details

- **Chain**: Arbitrum Sepolia (Chain ID: 421614)
- **Pool Contract**: `0x83EC719A6F504583d0F88CEd111cB8e8c0956431`
- **Supported Tokens**: MUSDC-A, MUSDC-B, MUSDC-C, MUSDC-D, MUSDC-E
