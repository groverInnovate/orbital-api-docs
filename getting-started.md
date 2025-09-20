# Getting Started

This guide will help you integrate the Orbital Pool API into your application in under 10 minutes.

## Prerequisites

- Basic knowledge of REST APIs and JSON
- A wallet with Arbitrum Sepolia ETH for gas fees
- Some test tokens (MUSDC-A, MUSDC-B, etc.) for testing

## Step 1: Test the API

First, verify the API is accessible:

```bash
curl https://your-orbital-api.com/health
```

**Expected Response:**
```json
{
  "status": "healthy",
  "pool_address": "0x83EC719A6F504583d0F88CEd111cB8e8c0956431",
  "network": "Arbitrum Sepolia",
  "chain_id": 421614
}
```

## Step 2: Get Supported Tokens

Fetch the list of supported tokens:

```bash
curl https://your-orbital-api.com/tokens
```

**Response:**
```json
{
  "tokens": {
    "0": {"address": "0x9666526dcF585863f9ef52D76718d810EE77FB8D", "symbol": "MUSDC-A"},
    "1": {"address": "0x1921d350666BA0Cf9309D4DA6a033EE0f0a70bEC", "symbol": "MUSDC-B"},
    "2": {"address": "0x...", "symbol": "MUSDC-C"},
    "3": {"address": "0x...", "symbol": "MUSDC-D"},
    "4": {"address": "0x...", "symbol": "MUSDC-E"}
  },
  "pool_address": "0x83EC719A6F504583d0F88CEd111cB8e8c0956431"
}
```

## Step 3: Execute Your First Swap

### 3.1 Prepare the Swap Request

```javascript
const swapRequest = {
  token_in_index: 0,        // MUSDC-A
  token_out_index: 1,       // MUSDC-B
  amount_in: "1000000000000000000",  // 1 token (18 decimals)
  min_amount_out: "0",      // Accept any amount (for testing)
  user_address: "0xYourWalletAddress"
};
```

### 3.2 Call the Swap API

```javascript
const response = await fetch('https://your-orbital-api.com/swap', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json'
  },
  body: JSON.stringify(swapRequest)
});

const swapData = await response.json();
```

### 3.3 Handle the Response

**Success Response:**
```json
{
  "success": true,
  "token_in": {
    "address": "0x9666526dcF585863f9ef52D76718d810EE77FB8D",
    "symbol": "MUSDC-A"
  },
  "token_out": {
    "address": "0x1921d350666BA0Cf9309D4DA6a033EE0f0a70bEC",
    "symbol": "MUSDC-B"
  },
  "transaction_data": {
    "to": "0x83EC719A6F504583d0F88CEd111cB8e8c0956431",
    "data": "0x5673b02d...",
    "gasLimit": 300000,
    "gasPrice": "100000000",
    "value": 0,
    "chainId": 421614
  },
  "gas_estimate": 300000,
  "gas_price": "100000000"
}
```

## Step 4: Sign and Send Transaction

### Using ethers.js

```javascript
import { ethers } from 'ethers';

// Connect to wallet
const provider = new ethers.providers.Web3Provider(window.ethereum);
const signer = provider.getSigner();

// Send transaction
const txResponse = await signer.sendTransaction(swapData.transaction_data);
console.log('Transaction sent:', txResponse.hash);

// Wait for confirmation
const receipt = await txResponse.wait();
console.log('Transaction confirmed in block:', receipt.blockNumber);
```

### Using web3.js

```javascript
import Web3 from 'web3';

const web3 = new Web3(window.ethereum);
const accounts = await web3.eth.getAccounts();

const txHash = await web3.eth.sendTransaction({
  from: accounts[0],
  ...swapData.transaction_data
});

console.log('Transaction sent:', txHash);
```

## Step 5: Handle Errors

Always implement proper error handling:

```javascript
try {
  const response = await fetch('https://your-orbital-api.com/swap', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(swapRequest)
  });

  if (!response.ok) {
    throw new Error(`HTTP ${response.status}: ${response.statusText}`);
  }

  const data = await response.json();
  
  if (!data.success) {
    throw new Error(data.error || 'Swap failed');
  }

  // Process successful response
  const txResponse = await signer.sendTransaction(data.transaction_data);
  
} catch (error) {
  console.error('Swap failed:', error.message);
  // Handle error appropriately
}
```

## Integration Patterns

### Frontend Integration

Perfect for:
- **DeFi dApps** - Add swap functionality
- **Wallets** - Integrate DEX features  
- **Portfolio Managers** - Rebalancing tools

```javascript
class OrbitalIntegration {
  constructor(apiUrl) {
    this.apiUrl = apiUrl;
  }

  async swap(tokenIn, tokenOut, amountIn, userAddress) {
    const response = await fetch(`${this.apiUrl}/swap`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        token_in_index: tokenIn,
        token_out_index: tokenOut,
        amount_in: amountIn,
        min_amount_out: '0',
        user_address: userAddress
      })
    });
    
    return response.json();
  }
}
```

### Backend Integration

Perfect for:
- **Trading Bots** - Automated strategies
- **Arbitrage** - Cross-DEX opportunities
- **Market Making** - Liquidity provision

```python
import requests
from web3 import Web3

class OrbitalBot:
    def __init__(self, api_url, private_key):
        self.api_url = api_url
        self.w3 = Web3(Web3.HTTPProvider('https://sepolia-rollup.arbitrum.io/rpc'))
        self.account = self.w3.eth.account.from_key(private_key)
    
    def execute_swap(self, token_in, token_out, amount):
        # Get transaction data
        response = requests.post(f'{self.api_url}/swap', json={
            'token_in_index': token_in,
            'token_out_index': token_out,
            'amount_in': str(amount),
            'min_amount_out': '0',
            'user_address': self.account.address
        })
        
        swap_data = response.json()
        
        # Sign and send transaction
        signed_txn = self.w3.eth.account.sign_transaction(
            swap_data['transaction_data'], 
            private_key=self.account.key
        )
        
        tx_hash = self.w3.eth.send_raw_transaction(signed_txn.rawTransaction)
        return tx_hash.hex()
```

## Common Gotchas

### 1. Token Approvals

Before swapping, ensure the token is approved:

```javascript
// Check allowance
const tokenContract = new ethers.Contract(tokenAddress, ERC20_ABI, signer);
const allowance = await tokenContract.allowance(userAddress, poolAddress);

// Approve if needed
if (allowance.lt(amountIn)) {
  const approveTx = await tokenContract.approve(poolAddress, amountIn);
  await approveTx.wait();
}
```

### 2. Gas Estimation

The API provides gas estimates, but you may want to add a buffer:

```javascript
const gasLimit = Math.floor(swapData.gas_estimate * 1.2); // 20% buffer
const txData = {
  ...swapData.transaction_data,
  gasLimit: gasLimit
};
```

### 3. Slippage Protection

Always set appropriate `min_amount_out` for production:

```javascript
const expectedOut = await getExpectedOutput(tokenIn, tokenOut, amountIn);
const minAmountOut = expectedOut * 0.95; // 5% slippage tolerance
```

## Next Steps

- [**API Reference**](./api-reference.md) - Complete endpoint documentation
- [**Integration Guides**](./integration-guides.md) - Advanced integration patterns
- [**Examples**](./examples.md) - Real-world code samples
- [**Error Handling**](./error-handling.md) - Comprehensive error guide

## Need Help?

- 📧 **Email**: support@orbital.com
- 💬 **Discord**: [Orbital Community](https://discord.gg/orbital)
- 🐛 **Issues**: [GitHub Issues](https://github.com/your-org/orbital-pool/issues)
- 📖 **Docs**: You're reading them!

---

**Next**: [API Reference →](./api-reference.md)
