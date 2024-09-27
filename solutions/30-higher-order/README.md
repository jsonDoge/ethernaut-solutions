# Answer

Goal: Set HigherOrder contract `commander` address to own wallet address.

## 1. Faulty assignment

The `registerTreasury()` has one main vulnerability which allows the exploit to happen. Due to using the older solidity version `0.6.12` types are not checked. So even though `registerTreasury(uint8)` should only allow 0x0-0xff values, it doesn't stop us from providing larger ones.

We can craft a transaction likes this:

```javascript
sendTransaction({
    to: contract.address,
    from: player,
    data: '211c85ab000000000000000000000000000000000000000000000000000000000000ffff'
})
```

The data field is invalid because we are passing 0xffff as an argument, which exceeds uint8. Yet the transaction succeeds and sets the total value to `treasury`.

## 2. Becoming `commander`

From here we only need to submit a call to `claimLeadership`:

```javascript
await contract.claimLeadership()
```
