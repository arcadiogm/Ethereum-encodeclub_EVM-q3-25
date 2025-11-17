# Lesson 11 - Tests for TokenSale.sol

## (Review) Challenge explanation

* Smart Contract Features
  * Buy ERC20 tokens with ETH for a fixed ratio
    * Ratio _r_ means that 1 ETH should buy _r_ tokens
  * Withdraw ETH by burning the ERC20 tokens at the contract
  * Mint a new ERC721 for a configured price
    * Price _p_ means that 1 NFT should cost _p_ tokens
  * Allow users to burn their NFTs to recover half of the purchase price
  * Update owner withdrawable amount whenever a NFT is sold
  * Allow owner to withdraw tokens from the contract
    * Only half of sales value is available for withdraw
* Architecture overview
* Contract external calls

## Completing tests

* (Review) Best practices on external calls
* (Review) Dealing with decimals and divisions
* (Review) Patterns

## References

<https://docs.soliditylang.org/en/latest/types.html#division>

<https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/BigInt>

<https://fravoll.github.io/solidity-patterns/>

<https://docs.soliditylang.org/en/latest/common-patterns.html>

<https://dev.to/jamiescript/design-patterns-in-solidity-1i28>


