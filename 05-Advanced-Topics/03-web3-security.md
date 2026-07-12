# Web3 Security: Blockchain & DeFi Auditing

## Smart Contract Vulnerabilities

### Reentrancy
```solidity
function withdraw(uint amount) public {
    require(balances[msg.sender] >= amount);
    (bool success, ) = msg.sender.call{value: amount}("");
    require(success);
    balances[msg.sender] -= amount;  // State update AFTER external call
}
```
Exploit: Flash loan -> deposit -> withdraw -> re-enter -> drain

### Oracle Manipulation
```solidity
uint price = uniswapV2Pool.getReserves();  // Single source, manipulable
```
Chain: Flash borrow -> swap to manipulate pool -> protocol reads manipulated price -> exploit

### Access Control
```solidity
function mint(address to, uint amount) public {
    _mint(to, amount);  // Missing onlyOwner
}
```

### Flash Loan Attack Pattern
1. Borrow large amount from flash loan provider
2. Manipulate AMM price / inflate share price
3. Extract value from protocol
4. Repay loan + fee

### DeFi Bugs
- Donation attacks: direct transfer inflates share price
- Governance attacks: flash loan voting power
- Precision loss: rounding errors in share calculation
- Liquidation flaws: self-liquidation at profit

## Bug Bounty Landscape

### Platforms
- Immunefi: $115M+ paid, primary DeFi platform
- HackenProof: Web3-focused
- HackerOne/Bugcrowd: growing blockchain programs

### Payouts
- Critical (fund loss): $10k-$1M+
- High (state manipulation): $5k-$50k
- Medium (logic flaw): $1k-$10k

## Tooling
```bash
slither contract.sol
forge test
mythril analyze contract.bin
echidna-test contract.sol --contract TestContract
```
