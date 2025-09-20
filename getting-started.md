# Getting Started

Basic integration guide for the Orbital Pool API.

## Prerequisites

- MetaMask wallet
- Arbitrum Sepolia ETH for gas
- Test tokens (MUSDC-A, MUSDC-B)

## Step 1: Check API Status

```bash
curl http://localhost:8000/health
```

**Response:**
```json
{
  "status": "healthy",
  "pool_address": "0x83EC719A6F504583d0F88CEd111cB8e8c0956431",
  "network": "Arbitrum Sepolia",
  "chain_id": 421614
}
```

## Step 2: Get Tokens

```bash
curl http://localhost:8000/tokens
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
  }
}
```

## Step 3: Make a Swap

### 3.1 Call the API

```javascript
const response = await fetch('http://localhost:8000/swap', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    token_in_index: 0,      // MUSDC-A
    token_out_index: 1,     // MUSDC-B  
    amount_in: "1000000000000000000", // 1 token
    min_amount_out: "0",
    user_address: "0xYourAddress"
  })
});

const swapData = await response.json();
```

### 3.2 API Response

```json
{
  "success": true,
  "transaction_data": {
    "to": "0x83EC719A6F504583d0F88CEd111cB8e8c0956431",
    "data": "0x5673b02d...",
    "gasLimit": 300000,
    "gasPrice": "100000000",
    "chainId": 421614
  }
}
```

### 3.3 Send Transaction

```javascript
// Using ethers.js
const provider = new ethers.providers.Web3Provider(window.ethereum);
const signer = provider.getSigner();

const tx = await signer.sendTransaction(swapData.transaction_data);
const receipt = await tx.wait();
console.log('Swap complete:', receipt.transactionHash);
```

## Error Handling

```javascript
if (!swapData.success) {
  console.error('Swap failed:', swapData.error);
  return;
}

try {
  const tx = await signer.sendTransaction(swapData.transaction_data);
  await tx.wait();
} catch (error) {
  console.error('Transaction failed:', error.message);
}
```

## Important Notes

### Token Approvals
Before swapping, approve the token:
```javascript
const tokenContract = new ethers.Contract(tokenAddress, ERC20_ABI, signer);
await tokenContract.approve(poolAddress, amountIn);
```

### Network
Make sure you're on Arbitrum Sepolia (Chain ID: 421614)

### Gas
The API estimates gas, but transactions may still fail if gas price changes

---

**Next**: [API Reference →](./api-reference.md)
