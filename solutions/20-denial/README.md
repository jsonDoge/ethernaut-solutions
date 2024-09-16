# Answer

Goal: Deny withdrawing funds when using `withdraw()`.

## 1. Injecting malicious address

The contract allows anyone to change `partner` value using `setWithdrawPartner()`. This address is later called in `withdraw()` and creates an opportunity for us.

```solidity
partner.call{value: amountToSend}("");
```

## 2. Drain gas

Due to the EIP-150 we can't drain all the gas, as the transaction will always keep 1/64th of the leftover gas after a `.call()`. So our goal is to drain as much of the 63/64th as possible. There are many ways to do this, like doing a lot of itteration or using SSTORE/SLOAD a lot. But in our mind the simple way is to create a reentrancy as `withdraw()` does not have protection against it.

We create a `MaliciousPartner` contract:

```solidity
contract MaliciousPartner {
    fallback() external payable {
        (msg.sender).call(abi.encodeWithSignature("withdraw()"));
    }
}
```

As you can see it simply calls back to `withdraw()`, since withdraw itself uses SSTORE it will do the job for us.

After deploying we set the new partner address:

```js
await contract.setWithdrawPartner('<MaliciousPartner address>')
```

## Caveat

This is not a perfect solution as `withdraw()` would still succeed if ~1.9M gas was provided. At such high gas limits the 1/64th part becomes large enough for the transaction to finish. But since the task explicitly says up to 1M gas, this solution is just enough.
