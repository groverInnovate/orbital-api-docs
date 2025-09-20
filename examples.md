# Examples

Real-world code examples for integrating Orbital Pool API.

## Basic Swap Example

### JavaScript/ethers.js

```javascript
import { ethers } from 'ethers';

async function swapTokens() {
  // Connect to wallet
  const provider = new ethers.providers.Web3Provider(window.ethereum);
  await provider.send('eth_requestAccounts', []);
  const signer = provider.getSigner();
  const userAddress = await signer.getAddress();

  // Swap 1 MUSDC-A for MUSDC-B
  const response = await fetch('http://localhost:8000/swap', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      token_in_index: 0,
      token_out_index: 1,
      amount_in: ethers.utils.parseEther('1').toString(),
      min_amount_out: '0',
      user_address: userAddress
    })
  });

  const data = await response.json();
  
  if (data.success) {
    const tx = await signer.sendTransaction(data.transaction_data);
    console.log('Swap successful:', tx.hash);
    return await tx.wait();
  }
}
```

### Python/web3.py

```python
from web3 import Web3
import requests

def swap_tokens(private_key, amount_in):
    w3 = Web3(Web3.HTTPProvider('https://sepolia-rollup.arbitrum.io/rpc'))
    account = w3.eth.account.from_key(private_key)
    
    # Get swap transaction
    response = requests.post('http://localhost:8000/swap', json={
        'token_in_index': 0,
        'token_out_index': 1,
        'amount_in': str(amount_in),
        'min_amount_out': '0',
        'user_address': account.address
    })
    
    data = response.json()
    
    if data['success']:
        # Sign and send transaction
        signed_txn = w3.eth.account.sign_transaction(
            data['transaction_data'], 
            private_key
        )
        tx_hash = w3.eth.send_raw_transaction(signed_txn.rawTransaction)
        return w3.eth.wait_for_transaction_receipt(tx_hash)
```

## Trading Bot Example

```javascript
class OrbitalTradingBot {
  constructor(apiUrl, privateKey) {
    this.apiUrl = apiUrl;
    this.provider = new ethers.providers.JsonRpcProvider(
      'https://sepolia-rollup.arbitrum.io/rpc'
    );
    this.wallet = new ethers.Wallet(privateKey, this.provider);
  }

  async checkArbitrage() {
    const amount = ethers.utils.parseEther('1');
    
    // Check A->B->A cycle
    const swapAB = await this.getQuote(0, 1, amount);
    const swapBA = await this.getQuote(1, 0, swapAB.expectedOut);
    
    const profit = swapBA.expectedOut.sub(amount);
    const profitPercent = profit.mul(100).div(amount);
    
    if (profitPercent.gt(5)) { // 5% profit threshold
      console.log(`Arbitrage opportunity: ${profitPercent}% profit`);
      await this.executeArbitrage(0, 1, amount);
    }
  }

  async executeArbitrage(tokenA, tokenB, amount) {
    try {
      // Execute A -> B
      await this.swap(tokenA, tokenB, amount);
      
      // Get new balance
      const balance = await this.getTokenBalance(tokenB);
      
      // Execute B -> A
      await this.swap(tokenB, tokenA, balance);
      
      console.log('Arbitrage completed successfully');
    } catch (error) {
      console.error('Arbitrage failed:', error);
    }
  }
}
```

## DeFi Portfolio Rebalancer

```typescript
interface PortfolioTarget {
  tokenIndex: number;
  targetPercent: number;
}

class PortfolioRebalancer {
  constructor(
    private apiUrl: string,
    private signer: ethers.Signer
  ) {}

  async rebalancePortfolio(targets: PortfolioTarget[]) {
    const userAddress = await this.signer.getAddress();
    
    // Get current balances
    const balances = await this.getAllBalances(userAddress);
    const totalValue = this.calculateTotalValue(balances);
    
    // Calculate required swaps
    const swaps = [];
    
    for (const target of targets) {
      const currentValue = balances[target.tokenIndex];
      const targetValue = totalValue.mul(target.targetPercent).div(100);
      const difference = targetValue.sub(currentValue);
      
      if (difference.abs().gt(ethers.utils.parseEther('0.01'))) {
        if (difference.gt(0)) {
          // Need to buy this token
          const sourceToken = this.findBestSource(balances, difference);
          swaps.push({
            from: sourceToken,
            to: target.tokenIndex,
            amount: difference
          });
        }
      }
    }
    
    // Execute swaps
    for (const swap of swaps) {
      await this.executeSwap(swap.from, swap.to, swap.amount);
    }
  }
}
```

---

**Next**: [Error Handling →](./error-handling.md)
