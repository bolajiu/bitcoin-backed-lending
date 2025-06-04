# BitVault - Secure Bitcoin-backed Lending Protocol

[![Stacks](https://img.shields.io/badge/Built%20on-Stacks-5546FF?style=flat-square)](https://stacks.co)
[![Clarity](https://img.shields.io/badge/Language-Clarity-orange?style=flat-square)](https://clarity-lang.org)

## Overview

BitVault is a decentralized lending protocol built on Stacks Layer 2 that enables Bitcoin holders to leverage their assets while maintaining self-custody principles. Users can deposit sBTC (Stacks Bitcoin) as collateral, borrow against it, and participate in liquidations to maintain system solvency.

### Key Features

- **Bitcoin-Native DeFi**: Built specifically for Bitcoin through Stacks Layer 2
- **Self-Custody**: Users maintain control of their assets throughout the process
- **Collateralized Lending**: Secure borrowing backed by sBTC deposits
- **Liquidation System**: Automated liquidation mechanism to maintain protocol solvency
- **Transparent Governance**: On-chain parameter management and protocol controls

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    BitVault Protocol                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────────┐    ┌─────────────────┐                │
│  │   User Deposits │    │   User Borrows  │                │
│  │                 │    │                 │                │
│  │ • Amount        │    │ • Amount        │                │
│  │ • Principal     │    │ • Collateral    │                │
│  └─────────────────┘    └─────────────────┘                │
│                                                             │
│  ┌─────────────────┐    ┌─────────────────┐                │
│  │ Liquidation     │    │ Protocol State  │                │
│  │ System          │    │                 │                │
│  │                 │    │ • Total Deposits│                │
│  │ • Threshold     │    │ • Total Borrows │                │
│  │ • Rewards       │    │ • Interest Rate │                │
│  │ • Automation    │    │ • Owner/Paused  │                │
│  └─────────────────┘    └─────────────────┘                │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│                     SIP-010 Token Interface                │
├─────────────────────────────────────────────────────────────┤
│                        Stacks Layer 2                      │
├─────────────────────────────────────────────────────────────┤
│                       Bitcoin Network                      │
└─────────────────────────────────────────────────────────────┘
```

## Protocol Parameters

| Parameter | Value | Description |
|-----------|-------|-------------|
| **Minimum Collateral Ratio** | 150% | Minimum collateralization required for borrowing |
| **Liquidation Threshold** | 80% | Default threshold for position liquidation |
| **Interest Rate Range** | 1% - 100% | Configurable annual percentage rate |
| **Liquidation Bonus** | 5% | Reward for liquidators |
| **Maximum Collateral Seizure** | 50% | Maximum collateral that can be seized in liquidation |

## Core Functions

### User Operations

#### Deposit Collateral

```clarity
(deposit-collateral token-contract amount)
```

Deposit sBTC tokens as collateral to enable borrowing.

**Parameters:**

- `token-contract`: SIP-010 compliant token contract
- `amount`: Amount of tokens to deposit

#### Borrow

```clarity
(borrow token-contract amount)
```

Borrow against deposited collateral, subject to minimum collateralization ratio.

**Parameters:**

- `token-contract`: SIP-010 compliant token contract  
- `amount`: Amount to borrow

#### Repay

```clarity
(repay token-contract amount)
```

Repay borrowed amount to reduce debt and improve collateralization ratio.

**Parameters:**

- `token-contract`: SIP-010 compliant token contract
- `amount`: Amount to repay

### Liquidation System

#### Liquidate Position

```clarity
(liquidate token-contract user amount)
```

Liquidate an under-collateralized position and earn rewards.

**Parameters:**

- `token-contract`: SIP-010 compliant token contract
- `user`: Principal of the user to liquidate
- `amount`: Amount of debt to liquidate

#### Claim Rewards

```clarity
(claim-rewards token-contract)
```

Claim accumulated liquidation rewards.

## Risk Management

### Collateralization Requirements

- **Minimum Ratio**: 150% collateralization required for all borrows
- **Liquidation Threshold**: Positions become liquidatable at 80% collateralization
- **Safety Buffer**: 70 percentage point buffer between minimum and liquidation threshold

### Liquidation Mechanics

1. **Trigger**: Position falls below 80% collateralization ratio
2. **Liquidator Action**: Anyone can liquidate the position
3. **Reward**: Liquidator receives 5% bonus (max 50% of collateral)
4. **Remaining Collateral**: Returns to original borrower after liquidation

## Administrative Controls

### Interest Rate Management

```clarity
(set-interest-rate new-rate)
```

Update the protocol's interest rate (1-100% APR).

### Liquidation Threshold Updates

```clarity
(set-liquidation-threshold new-threshold)  
```

Modify liquidation threshold (70-95% range).

### Emergency Controls

```clarity
(pause-protocol)
(unpause-protocol)
```

Pause/resume protocol operations for emergency situations.

## Read-Only Functions

### User Information

- `get-user-deposits`: Retrieve user's deposit information
- `get-user-borrows`: Get user's borrowing details and collateral

### Protocol Statistics

- `get-protocol-stats`: View total deposits, borrows, and interest rate

## Error Codes

| Code | Constant | Description |
|------|----------|-------------|
| 100 | `ERR-NOT-AUTHORIZED` | Insufficient permissions |
| 101 | `ERR-INSUFFICIENT-BALANCE` | Insufficient token balance |
| 102 | `ERR-INSUFFICIENT-COLLATERAL` | Collateral below minimum requirement |
| 103 | `ERR-INVALID-AMOUNT` | Invalid amount specified |
| 104 | `ERR-ALREADY-INITIALIZED` | Contract already initialized |
| 105 | `ERR-NOT-INITIALIZED` | Contract not initialized |
| 106 | `ERR-LIQUIDATION-FAILED` | Liquidation attempt failed |

## Security Features

### Safe Arithmetic

- Overflow protection in multiplication operations
- Underflow prevention in subtraction operations
- Input validation for all mathematical operations

### Access Control

- Contract owner verification for administrative functions
- Token contract validation
- Protocol pause mechanism for emergency situations

### Input Validation

- Non-zero amount requirements
- Collateralization ratio enforcement
- Parameter range validation

## Integration Guide

### Prerequisites

1. Deploy on Stacks testnet/mainnet
2. Configure allowed sBTC token contract
3. Initialize protocol with proper parameters

### Basic Integration Example

```javascript
// Deposit collateral
await contractCall({
  contractAddress: 'SP...',
  contractName: 'bitvault',
  functionName: 'deposit-collateral',
  functionArgs: [
    contractPrincipalCV('SP...', 'sbtc-token'),
    uintCV(1000000) // 1 sBTC (assuming 6 decimals)
  ]
});

// Borrow against collateral
await contractCall({
  contractAddress: 'SP...',
  contractName: 'bitvault', 
  functionName: 'borrow',
  functionArgs: [
    contractPrincipalCV('SP...', 'sbtc-token'),
    uintCV(600000) // Borrow 0.6 sBTC (150% collateralization)
  ]
});
```

## Development Setup

### Prerequisites

- [Clarinet](https://github.com/hirosystems/clarinet) for local development
- Stacks CLI for deployment
- Node.js for frontend integration

### Local Testing

```bash
# Clone repository
git clone https://github.com/bolajiu/bitcoin-backed-lending.git
cd bitcoin-backed-lending

# Run tests
clarinet test

# Start local devnet
clarinet integrate
```

## Roadmap

- [ ] **Phase 1**: Core lending functionality ✅
- [ ] **Phase 2**: Interest rate automation based on utilization
- [ ] **Phase 3**: Multi-collateral support (STX, other SIP-010 tokens)
- [ ] **Phase 4**: Governance token and DAO implementation
- [ ] **Phase 5**: Cross-chain collateral integration

## Contributing

We welcome contributions to BitVault! Please see our [Contributing Guidelines](CONTRIBUTING.md) for details on how to submit pull requests, report issues, and contribute to the codebase.

### Development Guidelines

1. All functions must include comprehensive error handling
2. Follow Clarity best practices for security
3. Include unit tests for new functionality
4. Update documentation for any API changes
