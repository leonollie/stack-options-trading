# Stacks Options Trading Platform - Smart Contract Documentation

![Options Trading](https://img.shields.io/badge/Type-Decentralized_Finance-blueviolet)
![Network](https://img.shields.io/badge/Network-StacksTestnet-orange)

## Overview

A decentralized options trading platform enabling secure creation, trading, and settlement of CALL/PUT options contracts. Built on Clarity with SIP-010 token compatibility and oracle price integration.

## Key Features

- **Option Types**: Supports both CALL (right to buy) and PUT (right to sell) options
- **Collateral Management**: Automated collateral locking/validation based on strike price
- **SIP-010 Integration**: Works with any compliant token for premiums/collateral
- **Price Oracle**: Real-time market data integration for fair settlement
- **Protocol Fees**: Configurable fee structure for sustainable operations
- **Position Tracking**: User-friendly interface to track open positions
- **Admin Controls**: Whitelisting for tokens/price symbols and security parameters

---

## Contract Architecture

### Data Structures

#### `options` Map

```clarity
{
    writer: principal,           // Option creator
    holder: (optional principal),// Current owner
    collateral-amount: uint,     // Locked collateral
    strike-price: uint,          // Exercise price
    premium: uint,               // Option cost
    expiry: uint,                // Expiration block
    is-exercised: bool,          // Exercise status
    option-type: "CALL"|"PUT",   // Contract type
    state: "ACTIVE"|"EXERCISED"  // Lifecycle state
}
```

#### `user-positions` Map

Tracks user engagement:

- Written options list (max 10)
- Held options list (max 10)
- Total collateral locked

---

## Core Functions

### 1. Option Lifecycle

#### `write-option`

Creates new option contract

```clarity
(define-public (write-option
    (token <sip-010-trait>)      // Collateral token
    (collateral-amount uint)     // Must meet strike requirements
    (strike-price uint)          // Exercise trigger price
    (premium uint)               // Purchase cost
    (expiry uint)                // Block height expiration
    (option-type (string-ascii 4)) // "CALL"/"PUT"
)
```

#### `buy-option`

Purchases existing option

```clarity
(define-public (buy-option
    (token <sip-010-trait>)      // Premium payment token
    (option-id uint)             // Target option ID
)
```

#### `exercise-option`

Executes option contract

```clarity
(define-public (exercise-option
    (token <sip-010-trait>)      // Settlement token
    (option-id uint)             // Target option ID
)
```

### 2. Oracle Integration

#### Price Feed Structure

```clarity
{
    price: uint,         // Current asset price (USD*1e6)
    timestamp: uint,     // Last update time
    source: principal    // Trusted oracle address
}
```

#### `update-price-feed` (Admin)

```clarity
(define-public (update-price-feed
    (symbol (string-ascii 10))   // Market symbol (e.g., "BTC-USD")
    (price uint)                 // Scaled price value
    (timestamp uint)             // Valid time reference
)
```

---

## Administrative Controls

### Token Management

#### `set-approved-token`

```clarity
(define-public (set-approved-token
    (token principal)    // SIP-010 contract address
    (approved bool)      // Whitelist status
)
```

### Symbol Management

#### `set-allowed-symbol`

```clarity
(define-public (set-allowed-symbol
    (symbol (string-ascii 10)) // Price symbol
    (allowed bool)              // Permission status
)
```

### Fee Configuration

#### `set-protocol-fee-rate`

```clarity
(define-public (set-protocol-fee-rate
    (new-rate uint) // Basis points (1% = 100)
)
```

---

## Error Codes

| Code  | Description             |
| ----- | ----------------------- |
| u1000 | Unauthorized operation  |
| u1001 | Insufficient balance    |
| u1002 | Invalid expiration time |
| u1003 | Invalid strike price    |
| u1004 | Option not found        |
| u1005 | Option expired          |
| u1006 | Insufficient collateral |
| u1007 | Already exercised       |
| u1008 | Invalid premium amount  |
| u1009 | Unapproved token        |
| u1010 | Invalid price symbol    |

---

## Security Model

1. **Collateral Verification**

   - CALL: Collateral ≥ Strike Price
   - PUT: Collateral ≥ (Strike Price / Current Price)

2. **Critical Asset Protection**

   - Core tokens (wBTC, wSTX) cannot be removed from whitelist

3. **Oracle Safeguards**

   - Price updates require valid future timestamps
   - Only pre-approved market symbols accepted

4. **User Protections**
   - Zero-address prevention
   - Max 10 active positions per user
   - Expiry time must be future-dated

---

## Usage Examples

### Creating a CALL Option

1. Approve token transfer
2. Call `write-option` with:
   - Token: wBTC
   - Collateral: 1.5 BTC
   - Strike: $50,000
   - Expiry: Block 150,000
   - Type: "CALL"

### Exercising a PUT Option

1. Verify current price < strike
2. Call `exercise-option` with:
   - Token: USDT
   - Option ID: 42

---

## Governance

- **Owner**: Initial deployer address (configurable)
- **Fee Structure**:
  - Default: 1% (100 basis points)
  - Max Cap: 10% (u1000 basis points)
