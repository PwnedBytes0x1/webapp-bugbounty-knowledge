# Web3 / Smart Contract Security

## Attack Vectors

### Re-entrancy
```solidity
// Vulnerable: sends before updating state
function withdraw(uint256 amount) public {
    require(balances[msg.sender] >= amount);
    (bool success, ) = msg.sender.call{value: amount}("");
    require(success);
    balances[msg.sender] -= amount;
}

// Safe: update state first
function withdraw(uint256 amount) public {
    require(balances[msg.sender] >= amount);
    balances[msg.sender] -= amount;
    (bool success, ) = msg.sender.call{value: amount}("");
    require(success);
}
```

### Front-running
- Transaction ordering manipulation
- Sandwich attacks on DEX trades

### Flash Loan Attacks
- Oracle manipulation
- Price manipulation via large loans

### Other
- Integer overflow/underflow
- Access control issues (tx.origin vs msg.sender)
- Short address attack
- Signature replay attacks

## Web3 DApp Testing

### MetaMask Interaction
- Check how dApp interacts with wallet
- Test with different chain IDs
- Verify signature requests

### RPC Endpoint Testing
- Infura/Alchemy endpoint enumeration
- Rate limiting on RPC calls

### Common CVEs
| Vulnerability | Type | Severity |
|---------------|------|----------|
| Re-entrancy (DAO Hack 2016) | Smart Contract | Critical |
| Flash Loan Attack | DeFi | High |
| Oracle Manipulation | DeFi | High |
| Front-running | Transaction | Medium |