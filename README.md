# Automated Portfolio Management Protocol

A decentralized protocol for automated portfolio rebalancing and asset management implemented on Stacks Layer 2. This smart contract enables users to create, manage, and rebalance investment portfolios in a trustless and efficient manner.

## Features

- **Multi-Asset Portfolio Management**: Support for up to 10 tokens per portfolio
- **Automated Rebalancing**: Time-based portfolio rebalancing mechanism
- **Percentage-Based Allocation**: Precise control over asset allocation using basis points
- **Access Control**: Owner-based permissions for portfolio management
- **Protocol Fee System**: Built-in fee mechanism (0.25% default)
- **User Portfolio Tracking**: Efficient storage and retrieval of user portfolios

## Contract Architecture

### Constants

- `MAX-TOKENS-PER-PORTFOLIO`: Maximum number of tokens allowed per portfolio (10)
- `BASIS-POINTS`: Standard basis points scale for percentages (10000)
- Protocol fee: 25 basis points (0.25%)

### Data Structures

#### Portfolios Map

```clarity
{
    owner: principal,
    created-at: uint,
    last-rebalanced: uint,
    total-value: uint,
    active: bool,
    token-count: uint
}
```

#### Portfolio Assets Map

```clarity
{
    target-percentage: uint,
    current-amount: uint,
    token-address: principal
}
```

### Core Functions

#### Portfolio Creation

```clarity
(create-portfolio (initial-tokens (list 10 principal)) (percentages (list 10 uint)))
```

Creates a new portfolio with specified tokens and their target allocation percentages.

**Parameters:**

- `initial-tokens`: List of token principal addresses
- `percentages`: List of target percentages in basis points

**Requirements:**

- Token count must not exceed `MAX-TOKENS-PER-PORTFOLIO`
- Token and percentage lists must have matching lengths
- Sum of percentages must be valid
- At least two tokens must be provided

#### Portfolio Rebalancing

```clarity
(rebalance-portfolio (portfolio-id uint))
```

Triggers a rebalancing operation for the specified portfolio.

**Requirements:**

- Only portfolio owner can initiate rebalancing
- Portfolio must be active
- Updates last rebalanced timestamp

#### Update Portfolio Allocation

```clarity
(update-portfolio-allocation (portfolio-id uint) (token-id uint) (new-percentage uint))
```

Modifies the target allocation for a specific token in the portfolio.

**Requirements:**

- Only portfolio owner can update allocations
- New percentage must be valid (0-10000 basis points)
- Token ID must be valid

### Read-Only Functions

- `get-portfolio`: Retrieves portfolio details
- `get-portfolio-asset`: Gets specific asset details within a portfolio
- `get-user-portfolios`: Lists all portfolios owned by a user
- `calculate-rebalance-amounts`: Determines if rebalancing is needed

### Error Codes

| Code | Description          |
| ---- | -------------------- |
| 100  | Not authorized       |
| 101  | Invalid portfolio    |
| 102  | Insufficient balance |
| 103  | Invalid token        |
| 104  | Rebalance failed     |
| 105  | Portfolio exists     |
| 106  | Invalid percentage   |
| 107  | Max tokens exceeded  |
| 108  | Length mismatch      |
| 109  | User storage failed  |
| 110  | Invalid token ID     |

## Security Considerations

1. **Access Control**

   - Portfolio operations restricted to owners
   - Protocol owner functions properly segregated
   - No self-assignment in ownership transfers

2. **Input Validation**

   - Strict percentage validation (0-10000 basis points)
   - Token count limitations
   - List length verification

3. **State Management**
   - Atomic operations for portfolio updates
   - Safe storage of user portfolio lists
   - Active status tracking

## Usage Examples

### Creating a Portfolio

```clarity
;; Create a portfolio with two tokens
(create-portfolio
    (list
        'SP2PABAF9FTAJYNFZH93XENAJ8FVY99RRM50D2JG9.token-a
        'SP2PABAF9FTAJYNFZH93XENAJ8FVY99RRM50D2JG9.token-b
    )
    (list
        u5000  ;; 50%
        u5000  ;; 50%
    )
)
```

### Updating Allocation

```clarity
;; Update token allocation
(update-portfolio-allocation
    u1  ;; portfolio-id
    u0  ;; token-id
    u6000  ;; New allocation: 60%
)
```

### Rebalancing

```clarity
;; Trigger portfolio rebalancing
(rebalance-portfolio u1)
```

## Integration Guidelines

1. **Portfolio Creation**

   - Ensure token contracts are deployed and accessible
   - Verify token list and percentage list match in length
   - Calculate percentages in basis points (multiply by 100)

2. **Rebalancing Strategy**

   - Monitor `last-rebalanced` timestamp
   - Use `calculate-rebalance-amounts` to check rebalancing needs
   - Handle rebalancing failures gracefully

3. **Error Handling**
   - Implement proper error catching for all contract calls
   - Validate inputs before sending transactions
   - Monitor transaction status for success/failure

## Protocol Fees

The protocol implements a fee system with the following characteristics:

- Default fee: 0.25% (25 basis points)
- Fees are calculated and collected during rebalancing operations
- Fee recipient is the protocol owner

## Future Improvements

1. **Enhanced Features**

   - Dynamic fee adjustment mechanism
   - Support for more complex rebalancing strategies
   - Integration with external price oracles

2. **Optimization Opportunities**

   - Gas optimization for large portfolios
   - Batch processing for multiple portfolio operations
   - Enhanced event emission for better tracking

3. **Security Enhancements**
   - Additional access control layers
   - Emergency pause functionality
   - Enhanced validation mechanisms
