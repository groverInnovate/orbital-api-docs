# Orbital Pool API Documentation

Welcome to the Orbital Pool API documentation. The Orbital Pool API provides developers with simple REST endpoints to integrate with the revolutionary 5-token AMM protocol powered by torus-based invariant mathematics.

## 🚀 Quick Start

```bash
# Get pool health status
curl https://your-orbital-api.com/health

# Execute a token swap
curl -X POST https://your-orbital-api.com/swap \
  -H "Content-Type: application/json" \
  -d '{
    "token_in_index": 0,
    "token_out_index": 1,
    "amount_in": "1000000000000000000",
    "min_amount_out": "0",
    "user_address": "0x..."
  }'
```

## 📚 Documentation Sections

- [**Overview**](./overview.md) - Introduction and key concepts
- [**Getting Started**](./getting-started.md) - Quick integration guide
- [**API Reference**](./api-reference.md) - Complete endpoint documentation
- [**Integration Guides**](./integration-guides.md) - Step-by-step tutorials
- [**Examples**](./examples.md) - Code samples and use cases
- [**Error Handling**](./error-handling.md) - Troubleshooting guide

## 🌟 Key Features

- **5-Token AMM** - Revolutionary torus-based invariant for optimal price discovery
- **Simple REST API** - Easy integration with any programming language
- **Gas Optimization** - Built-in gas estimation and optimization
- **Secure Design** - Never handles private keys, returns unsigned transactions
- **Production Ready** - Battle-tested on Arbitrum Sepolia

## 🔗 Quick Links

- [Live API](https://your-orbital-api.com) - Production endpoint
- [GitHub Repository](https://github.com/your-org/orbital-pool) - Source code
- [Demo Application](https://orbital-demo.com) - Interactive example
- [Support](mailto:support@orbital.com) - Get help

---

*Built with ❤️ for the DeFi community*
