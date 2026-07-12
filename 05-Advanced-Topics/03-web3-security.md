# Web3 Security: 2026 Reference

## Smart Contract Vulnerabilities

| Vulnerability | Description | Impact |
|---------------|-------------|--------|
| Reentrancy | External call before state update | Theft of funds |
| Flash loan attacks | Unc-collateralized loan + price manipulation | Protocol drain |
| Oracle manipulation | Manipulating price feed input | False pricing |
| Access control | Missing onlyOwner modifier | Admin-level attacks |
| Integer overflow | Arithmetic without SafeMath | Balance manipulation |
| Front-running | TX ordering manipulation | MEV extraction |

## Common Attack Patterns

### DeFi
- Price oracle manipulation (TWAP vs spot)
- Liquidity pool draining via sandwich attacks
- Governance token takeover (flash loan + proposal)
- Cross-chain bridge attacks (wormhole, nomad patterns)

### NFT
- Floor price manipulation
- Trait-based rarity exploitations
- Phishing via malicious marketplace listings
- Royalty bypass

## Testing Tools
- **Foundry**: Fast Rust-based Solidity testing framework
- **Slither**: Static analysis for Solidity
- **Echidna**: Fuzzing for Solidity
- **Mythril**: Security analysis for EVM bytecode

## Web3 Bug Bounty Platforms
- Immunefi (largest Web3 bug bounty)
- Hats Finance (decentralized)