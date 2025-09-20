# API Reference

Complete reference for all Orbital Pool API endpoints.

## Base URL

```
http://localhost:8000
```

## Authentication

No authentication required. All endpoints are public.

---

## Endpoints

### Health Check

Check API and pool status.

```http
GET /health
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

**Response Fields:**
- `status` - API health status (`healthy` | `degraded` | `down`)
- `pool_address` - Orbital Pool contract address
- `network` - Blockchain network name
- `chain_id` - Network chain ID

---

### Get Supported Tokens

Retrieve all supported tokens and their information.

```http
GET /tokens
```

**Response:**
```json
{
  "tokens": {
    "0": {
      "address": "0x9666526dcF585863f9ef52D76718d810EE77FB8D",
      "symbol": "MUSDC-A"
    },
    "1": {
      "address": "0x1921d350666BA0Cf9309D4DA6a033EE0f0a70bEC", 
      "symbol": "MUSDC-B"
    },
    "2": {
      "address": "0x...",
      "symbol": "MUSDC-C"
    },
    "3": {
      "address": "0x...",
      "symbol": "MUSDC-D"
    },
    "4": {
      "address": "0x...",
      "symbol": "MUSDC-E"
    }
  },
  "pool_address": "0x83EC719A6F504583d0F88CEd111cB8e8c0956431"
}
```

**Response Fields:**
- `tokens` - Object mapping token indices to token info
- `tokens[index].address` - Token contract address
- `tokens[index].symbol` - Token symbol
- `pool_address` - Orbital Pool contract address

---

### Execute Token Swap

Generate transaction data for swapping tokens.

```http
POST /swap
```

**Request Body:**
```json
{
  "token_in_index": 0,
  "token_out_index": 1,
  "amount_in": "1000000000000000000",
  "min_amount_out": "950000000000000000",
  "user_address": "0xd2D16c0c2c2c2c2c2c2c2c2c2c2c2c2c2c2c2c2c"
}
```

**Request Fields:**
- `token_in_index` *(integer)* - Input token index (0-4)
- `token_out_index` *(integer)* - Output token index (0-4)
- `amount_in` *(string)* - Input amount in wei (18 decimals)
- `min_amount_out` *(string)* - Minimum output amount in wei
- `user_address` *(string)* - User's wallet address (checksummed)

**Success Response (200):**
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
    "data": "0x5673b02d000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000010000000000000000000000000000000000000000000000000de0b6b3a76400000000000000000000000000000000000000000000000000000d2f13f7789f0000",
    "gasLimit": 300000,
    "gasPrice": "100000000",
    "value": 0,
    "chainId": 421614,
    "nonce": 42,
    "from": "0xd2D16c0c2c2c2c2c2c2c2c2c2c2c2c2c2c2c2c2c"
  },
  "gas_estimate": 300000,
  "gas_price": "100000000"
}
```

**Error Response (400):**
```json
{
  "success": false,
  "error": "Invalid token index"
}
```

**Response Fields:**
- `success` - Operation success status
- `token_in` - Input token information
- `token_out` - Output token information  
- `transaction_data` - Unsigned transaction object for signing
- `gas_estimate` - Estimated gas units required
- `gas_price` - Current gas price in wei

---

### Add Liquidity

Generate transaction data for adding liquidity to a tick.

```http
POST /liquidity/add
```

**Request Body:**
```json
{
  "k_value": 1000,
  "amounts": [
    "1000000000000000000",
    "1000000000000000000", 
    "1000000000000000000",
    "1000000000000000000",
    "1000000000000000000"
  ],
  "user_address": "0xd2D16c0c2c2c2c2c2c2c2c2c2c2c2c2c2c2c2c2c"
}
```

**Request Fields:**
- `k_value` *(integer)* - Tick identifier (k value)
- `amounts` *(array[string])* - Array of 5 token amounts in wei
- `user_address` *(string)* - User's wallet address

**Success Response (200):**
```json
{
  "success": true,
  "k_value": "1000",
  "amounts": [
    "1000000000000000000",
    "1000000000000000000",
    "1000000000000000000", 
    "1000000000000000000",
    "1000000000000000000"
  ],
  "transaction_data": {
    "to": "0x83EC719A6F504583d0F88CEd111cB8e8c0956431",
    "data": "0x...",
    "gasLimit": 400000,
    "gasPrice": "100000000",
    "value": 0,
    "chainId": 421614,
    "nonce": 43,
    "from": "0xd2D16c0c2c2c2c2c2c2c2c2c2c2c2c2c2c2c2c2c"
  },
  "gas_estimate": 400000,
  "gas_price": "100000000"
}
```

---

### Remove Liquidity

Generate transaction data for removing liquidity from a tick.

```http
POST /liquidity/remove
```

**Request Body:**
```json
{
  "k_value": 1000,
  "lp_shares": "500000000000000000",
  "user_address": "0xd2D16c0c2c2c2c2c2c2c2c2c2c2c2c2c2c2c2c2c"
}
```

**Request Fields:**
- `k_value` *(integer)* - Tick identifier (k value)
- `lp_shares` *(string)* - LP shares to remove in wei
- `user_address` *(string)* - User's wallet address

**Success Response (200):**
```json
{
  "success": true,
  "k_value": "1000",
  "lp_shares": "500000000000000000",
  "transaction_data": {
    "to": "0x83EC719A6F504583d0F88CEd111cB8e8c0956431",
    "data": "0x...",
    "gasLimit": 350000,
    "gasPrice": "100000000", 
    "value": 0,
    "chainId": 421614,
    "nonce": 44,
    "from": "0xd2D16c0c2c2c2c2c2c2c2c2c2c2c2c2c2c2c2c2c"
  },
  "gas_estimate": 350000,
  "gas_price": "100000000"
}
```

---

### Get Gas Price

Retrieve current network gas price.

```http
GET /gas-price
```

**Response:**
```json
{
  "gas_price": "100000000",
  "gas_price_gwei": "0.1"
}
```

**Response Fields:**
- `gas_price` - Gas price in wei
- `gas_price_gwei` - Gas price in Gwei for readability

---

## Error Responses

All endpoints return consistent error responses:

```json
{
  "success": false,
  "error": "Error description"
}
```

### Common Error Codes

| HTTP Status | Error | Description |
|-------------|-------|-------------|
| `400` | Bad Request | Invalid request parameters |
| `404` | Not Found | Endpoint not found |
| `500` | Internal Server Error | Server error |

### Specific Error Messages

#### Swap Errors
- `"Missing field: {field_name}"` - Required field missing
- `"Invalid token index"` - Token index not in range 0-4
- `"Cannot swap same token"` - Input and output tokens are identical
- `"Insufficient liquidity"` - Not enough liquidity for swap
- `"Amount too small"` - Input amount below minimum

#### Liquidity Errors  
- `"Must provide exactly 5 token amounts"` - Amounts array wrong length
- `"Invalid k value"` - K value out of valid range
- `"Insufficient LP shares"` - Not enough LP shares to remove

#### General Errors
- `"Invalid address format"` - Malformed Ethereum address
- `"Gas estimation failed"` - Unable to estimate gas
- `"Network error"` - Blockchain connection issue

---

**Next**: [Examples →](./examples.md)
