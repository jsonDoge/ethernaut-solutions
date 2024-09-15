# Answer

Goal: Claim ownership of the contract instance.

## 1. Causing dynamic array length underflow

After calling the `makeContact()` we are free to modify the codex as we desire using the provided functions.

```js
await contract.makeContact()
```

The function that should catch the eye is on line 23 - `retract()` which decreases the array length. But if the array is already 0 length it will cause an underflow and the length will become 0xfffff......

```js
await contract.retract()
```

## 2. Changing the owner

Now because the length has been maxed, we can use `revise()` to alter any index we want. And because the storage slot of a dynamic array elements is calculated as `keccak256(codexSlot) + elementIndex` we have access to all the storage slots.

The `codex` is stored in storage slot 1, because 0 is occupied by both `contact` boolean and `owner` address (packed together). So the 1st or \[0] element of `codex` will be stored at `keccak256(codexStorageSlot)` = `keccak256(<padded to 32>...000001)` = `0xb10e2d527612073b26eecdfd717e6a320cf44b4afac2b0732d9fcbe2b7fa0cf6`.

The question may arise - if `codex` \[0] element slot is starting that high, how do we go back to "0" storage slot where the owner is stored? The answers that after hitting the max storage slot key 0xfffff... by adding one more we'll be looping back to "0". So to find out which element index we need to alter `0xffffff... - keccak256(codexStorageSlot) + 1` = `0x4ef1d2ad89edf8c4d91132028e8195cdf30bb4b5053d4f8cd260341d4805f30a`.

Now that we know the index we only need to set it to our desired address value:

```js
await contract.revise(
  '0x4ef1d2ad89edf8c4d91132028e8195cdf30bb4b5053d4f8cd260341d4805f30a',
  '0x000000000000000000000001<new owner address>',
)
```

The "1" before the address is so that the `contact` would remain true (optional).
