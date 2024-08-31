# Answer

Goal: Claim ownership of the contract.

## 1. Becoming the owner

Contract has a misspelled constructor "Fal1out" instead of "Fallout" - the solution only requires the player to call it again:

```js
await contract.Fal1out();
```

