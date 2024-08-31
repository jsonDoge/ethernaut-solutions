# Answer

Goal: Gain a very large amount of tokens.

## 1. Gaining more tokens

The contract does not have underflow protection, which means if we manage to drop our balance below zero it will flip to max amount. This can be achieve, by calling:

```js
await contract.transfer('0x0000000000000000000000000000000000000000', 21)
```

