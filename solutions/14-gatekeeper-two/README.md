# Answer

Goal: Register as an entrant.

## 1. Passing gate one

The first is same as previous gatekeeper - modifier requires that the `msg.sender` and `tx.origin` would NOT be the same address. This means we'll be using a smart contract as a proxy to execute the exploit.

## 2. Passing gate two

The second modifier checks that the code stored at `caller()` is equal to 0. At first glance may seem that using contracts goes out the door, but if we call from within the contracts `constructor()` the `extcodesize()` will return 0.

## 3. Passing gate three

Final part is again a play on numbers, but this time it uses a bitwise XOR "^" operator, let's break it down:

1. `uint64(bytes8(keccak256(abi.encodePacked(msg.sender))))` is taking the msg.sender, hashing it and taking the 8 least significant bytes.

2. `^ uint64(_gateKey)` _gateKey is used with a bitwise XOR after which the result is expected to equal `type(uint64).max` which is all bits are 1.

To pass it we can use any `msg.sender` address and precalculcate the correct `_gateKey` so that the final operation would yield `type(uint64).max`.

## 4. Entering the smart contract

We end up with a contract that looks like this.

```solidity
contract GatekeeperTwoExploit {
    GatekeeperTwo g;

    constructor(address targetAddr) public {
        g = GatekeeperTwo(targetAddr);

        // Reversing the gateKey is simply applying XOR with type(uint64).max
        uint64 gateKey = uint64(bytes8(keccak256(abi.encodePacked(address(this))))) ^ type(uint64).max;

        g.enter(bytes8(gateKey));
    }
}
```
