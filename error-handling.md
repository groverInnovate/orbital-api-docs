# Error Handling & Troubleshooting

Comprehensive guide for handling errors and troubleshooting common issues.

## Error Response Format

All API endpoints return consistent error responses:

```json
{
  "success": false,
  "error": "Detailed error message"
}
```

## Common Errors

### API Errors

#### Invalid Parameters
```json
{
  "success": false,
  "error": "Missing field: token_in_index"
}
```

**Solution**: Ensure all required fields are provided in the request.

#### Invalid Token Index
```json
{
  "success": false,
  "error": "Invalid token index"
}
```

**Solution**: Use token indices 0-4 only.

#### Same Token Swap
```json
{
  "success": false,
  "error": "Cannot swap same token"
}
```

**Solution**: Ensure `token_in_index` ≠ `token_out_index`.

### Transaction Errors

#### Insufficient Balance
```
Error: insufficient funds for gas * price + value
```

**Solution**: Ensure wallet has enough ETH for gas fees.

#### Token Not Approved
```
Error: ERC20: transfer amount exceeds allowance
```

**Solution**: Approve tokens before swapping:

```javascript
const tokenContract = new ethers.Contract(tokenAddress, ERC20_ABI, signer);
await tokenContract.approve(poolAddress, amount);
```

#### Gas Estimation Failed
```json
{
  "success": false,
  "error": "Gas estimation failed"
}
```

**Solutions**:
- Check token balances
- Verify token approvals
- Ensure sufficient liquidity

## Error Handling Patterns

### Robust API Client

```javascript
class OrbitalClient {
  constructor(apiUrl, retries = 3) {
    this.apiUrl = apiUrl;
    this.retries = retries;
  }

  async makeRequest(endpoint, data, attempt = 1) {
    try {
      const response = await fetch(`${this.apiUrl}${endpoint}`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(data)
      });

      if (!response.ok) {
        throw new Error(`HTTP ${response.status}: ${response.statusText}`);
      }

      const result = await response.json();
      
      if (!result.success) {
        throw new OrbitalError(result.error, response.status);
      }

      return result;
    } catch (error) {
      if (attempt < this.retries && this.isRetryable(error)) {
        console.log(`Retrying request (${attempt}/${this.retries})`);
        await this.delay(1000 * attempt);
        return this.makeRequest(endpoint, data, attempt + 1);
      }
      throw error;
    }
  }

  isRetryable(error) {
    return error.status >= 500 || error.code === 'NETWORK_ERROR';
  }

  delay(ms) {
    return new Promise(resolve => setTimeout(resolve, ms));
  }
}

class OrbitalError extends Error {
  constructor(message, status) {
    super(message);
    this.name = 'OrbitalError';
    this.status = status;
  }
}
```

### Transaction Error Handling

```javascript
async function safeSwap(tokenIn, tokenOut, amountIn, signer) {
  try {
    // Pre-flight checks
    await validateSwapInputs(tokenIn, tokenOut, amountIn, signer);
    
    // Get transaction data
    const swapData = await orbital.getSwapTransaction({
      token_in_index: tokenIn,
      token_out_index: tokenOut,
      amount_in: amountIn,
      user_address: await signer.getAddress()
    });

    // Send transaction with error handling
    const tx = await signer.sendTransaction(swapData.transaction_data);
    
    // Wait for confirmation with timeout
    const receipt = await Promise.race([
      tx.wait(),
      new Promise((_, reject) => 
        setTimeout(() => reject(new Error('Transaction timeout')), 300000)
      )
    ]);

    return receipt;
  } catch (error) {
    throw new SwapError(error.message, error);
  }
}

async function validateSwapInputs(tokenIn, tokenOut, amountIn, signer) {
  const userAddress = await signer.getAddress();
  
  // Check ETH balance
  const ethBalance = await signer.provider.getBalance(userAddress);
  if (ethBalance.lt(ethers.utils.parseEther('0.001'))) {
    throw new Error('Insufficient ETH for gas fees');
  }

  // Check token balance
  const tokenContract = new ethers.Contract(
    TOKEN_ADDRESSES[tokenIn], 
    ERC20_ABI, 
    signer
  );
  const balance = await tokenContract.balanceOf(userAddress);
  if (balance.lt(amountIn)) {
    throw new Error('Insufficient token balance');
  }

  // Check allowance
  const allowance = await tokenContract.allowance(userAddress, POOL_ADDRESS);
  if (allowance.lt(amountIn)) {
    throw new Error('Token not approved. Please approve first.');
  }
}
```

## Troubleshooting Guide

### Connection Issues

**Problem**: API requests failing
```
Error: fetch failed
```

**Solutions**:
1. Check API URL is correct
2. Verify network connectivity
3. Check for CORS issues in browser
4. Ensure API is running and healthy

### MetaMask Issues

**Problem**: MetaMask not connecting
```
Error: window.ethereum is undefined
```

**Solutions**:
1. Install MetaMask extension
2. Unlock MetaMask wallet
3. Switch to correct network (Arbitrum Sepolia)
4. Refresh page and try again

**Problem**: Wrong network
```
Error: unsupported network
```

**Solution**: Switch to Arbitrum Sepolia:
```javascript
await window.ethereum.request({
  method: 'wallet_switchEthereumChain',
  params: [{ chainId: '0x66eee' }] // 421614 in hex
});
```

### Gas Issues

**Problem**: Transaction fails with "out of gas"
```
Error: transaction failed - out of gas
```

**Solutions**:
1. Increase gas limit by 20%
2. Check current gas prices
3. Retry during lower network congestion

**Problem**: Gas price too low
```
Error: replacement transaction underpriced
```

**Solution**: Use current gas price:
```javascript
const gasPrice = await provider.getGasPrice();
const txData = {
  ...swapData.transaction_data,
  gasPrice: gasPrice.mul(110).div(100) // 10% buffer
};
```

### Token Issues

**Problem**: Token transfer fails
```
Error: ERC20: transfer amount exceeds balance
```

**Solutions**:
1. Check actual token balance
2. Account for token decimals correctly
3. Ensure no pending transactions

**Problem**: Approval fails
```
Error: ERC20: approve to zero address
```

**Solution**: Use correct pool address:
```javascript
const POOL_ADDRESS = '0x83EC719A6F504583d0F88CEd111cB8e8c0956431';
await tokenContract.approve(POOL_ADDRESS, amount);
```

## Monitoring & Logging

### Transaction Monitoring

```javascript
class TransactionMonitor {
  constructor(provider) {
    this.provider = provider;
  }

  async monitorTransaction(txHash, timeout = 300000) {
    const startTime = Date.now();
    
    while (Date.now() - startTime < timeout) {
      try {
        const receipt = await this.provider.getTransactionReceipt(txHash);
        
        if (receipt) {
          if (receipt.status === 1) {
            console.log(`✅ Transaction confirmed: ${txHash}`);
            return receipt;
          } else {
            throw new Error(`❌ Transaction failed: ${txHash}`);
          }
        }
        
        // Check if transaction is still pending
        const tx = await this.provider.getTransaction(txHash);
        if (!tx) {
          throw new Error(`Transaction not found: ${txHash}`);
        }
        
        console.log(`⏳ Transaction pending: ${txHash}`);
        await this.delay(5000);
        
      } catch (error) {
        console.error(`Error monitoring transaction: ${error.message}`);
        throw error;
      }
    }
    
    throw new Error(`Transaction timeout: ${txHash}`);
  }

  delay(ms) {
    return new Promise(resolve => setTimeout(resolve, ms));
  }
}
```

### Error Logging

```javascript
class ErrorLogger {
  static log(error, context = {}) {
    const errorData = {
      timestamp: new Date().toISOString(),
      message: error.message,
      stack: error.stack,
      context,
      userAgent: navigator.userAgent,
      url: window.location.href
    };
    
    // Log to console
    console.error('Orbital Error:', errorData);
    
    // Send to monitoring service
    this.sendToMonitoring(errorData);
  }

  static sendToMonitoring(errorData) {
    // Send to your monitoring service (Sentry, LogRocket, etc.)
    fetch('/api/errors', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(errorData)
    }).catch(console.error);
  }
}
```

## Support Resources

### Getting Help

1. **Documentation**: Check this documentation first
2. **GitHub Issues**: [Report bugs](https://github.com/your-org/orbital-pool/issues)
3. **Discord**: [Community support](https://discord.gg/orbital)
4. **Email**: support@orbital.com

### Debug Information

When reporting issues, include:

```javascript
// Debug info to include
const debugInfo = {
  apiUrl: 'https://your-orbital-api.com',
  chainId: await provider.getNetwork().chainId,
  userAddress: await signer.getAddress(),
  blockNumber: await provider.getBlockNumber(),
  gasPrice: await provider.getGasPrice(),
  timestamp: new Date().toISOString(),
  userAgent: navigator.userAgent
};
```

### Health Checks

```javascript
async function systemHealthCheck() {
  const checks = {
    api: false,
    network: false,
    wallet: false,
    tokens: false
  };

  try {
    // API health
    const health = await fetch('https://your-orbital-api.com/health');
    checks.api = health.ok;

    // Network connectivity
    const provider = new ethers.providers.JsonRpcProvider(
      'https://sepolia-rollup.arbitrum.io/rpc'
    );
    await provider.getBlockNumber();
    checks.network = true;

    // Wallet connection
    if (window.ethereum) {
      const accounts = await window.ethereum.request({ 
        method: 'eth_accounts' 
      });
      checks.wallet = accounts.length > 0;
    }

    // Token contracts
    const tokens = await fetch('https://your-orbital-api.com/tokens');
    checks.tokens = tokens.ok;

  } catch (error) {
    console.error('Health check failed:', error);
  }

  return checks;
}
```

---

**🎉 Documentation Complete!**

You now have comprehensive documentation for the Orbital Pool API. The documentation includes:

- **Overview** - Introduction and key concepts
- **Getting Started** - Quick integration guide  
- **API Reference** - Complete endpoint documentation
- **Integration Guides** - Platform-specific tutorials
- **Examples** - Real-world code samples
- **Error Handling** - Troubleshooting guide

This professional documentation follows Uniswap's structure and provides everything developers need to integrate with your revolutionary 5-token AMM! 🚀
