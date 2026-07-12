# Web3 Security

Web3 security encompasses vulnerabilities in smart contracts, DeFi protocols, bridges, and blockchain infrastructure.

## Smart Contract Vulnerabilities

### Reentrancy
Calling an external contract that calls back into the original function before it completes.

```solidity
function withdraw(uint amount) public {
    require(balances[msg.sender] >= amount);
    (bool success, ) = msg.sender.call{value: amount}("");
    require(success);
    balances[msg.sender] -= amount;  // Updated AFTER external call
}
```

**Prevention**: Checks-effects-interactions pattern, reentrancy guards.

### Integer Overflow/Underflow
```solidity
// Solidity <0.8: No built-in overflow protection
function transfer(uint amount) public {
    balances[msg.sender] -= amount;  // Underflow: 0 - 1 = huge number
}
```

**Prevention**: Use Solidity 0.8+ (built-in checks) or SafeMath library.

### Flash Loan Attacks
- Borrow huge amounts without collateral in a single transaction
- Manipulate oracle prices, drain liquidity pools
- Common in AMMs (Uniswap, PancakeSwap)

### Oracle Manipulation
- Manipulating price feeds (Chainlink, Uniswap TWAP)
- Sandwich attacks around oracle updates
- Single-source oracle reliance

### Access Control
- Missing `onlyOwner` modifiers
- Initialization functions callable by anyone
- `tx.origin` for authentication (vulnerable to phishing)

### Front-Running / MEV
- Observing pending transactions and front-running
- Sandwich attacks on DEX trades
- Time-bandit attacks (reorgs on low-hash chains)

## DeFi Specific Issues

- **Impermanent loss** — Not a vulnerability but an economic risk
- **Liquidity pool manipulation** — Imbalance attacks
- **Yield aggregation risks** — Compounding strategy failures
- **Governance attacks** — Flash loan voting manipulation

## Bridge Vulnerabilities
- **Validator set compromise** — Bridge security relies on n validators
- **Message passing bugs** — Incorrect packet verification
- **Replay attacks** — Same message replayed on different chains
- **Honeypot bridges** — Fake bridges draining user funds

## Common Web3 Tools

| Tool | Purpose |
|------|---------|
| **Slither** | Static analysis for Solidity |
| **Mythril** | Security analysis for EVM bytecode |
| **Foundry** | Smart contract development + fuzzing |
| **Hardhat** | Development environment + testing |
| **Echidna** | Fuzzing framework |
| **Etherscan** | Contract verification and analysis |
| **Dune Analytics** | On-chain data analysis |

## Web3 Bug Bounty Programs

- **Immunefi** — Largest bug bounty platform for Web3
- **HackerOne web3 programs** — Some blockchain projects
- **Code4rena** — Competitive smart contract auditing
- **Sherlock** — Competitive audits with real bounties

---

> **Next**: [GraphQL Security](04-graphql-security.md)
