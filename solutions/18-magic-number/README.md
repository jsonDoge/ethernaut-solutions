# Answer

Goal: Deploy a contract with small code size which would respond to `whatIsTheMeaningOfLife()` with 42.

## 1. Small contract bytecode

We don't really need to implement the function with accurate function selector. The goal is for the contract to return 42, when called. So our bytecode looks like this:

```
0x602a5f5260205ff3
```

If we split it into each opcode we get this:

```
[00]	PUSH1	2a
[02]	PUSH0
[03]	MSTORE
[04]	PUSH1	20
[06]	PUSH0
[07]	RETURN
```

00-03 - stores the value 42 in hex (0x2a).
04-07 - return the value stored at 0x0 with size 0x20.

## 2. Deploying the smart contract

To deploy this bytecode we are going to write a small contract deployer so we could store this exact bytecode in the new contract:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract MagicNumberDeployer {
    event Address(address addr);

    function deploy() external {
        address contractAddress;
        assembly {
            let ptr := mload(0x40)
            // actual bytecode
            //          0x6026600c60003960206000f3602A5F5260205FF3<padding>
            mstore(ptr, 0x6008600c60003960086000f3602A5F5260205FF3000000000000000000000000)
            contractAddress := create(0, ptr, 20)
        }

        require(contractAddress != address(0), "Failed to deploy contract");

        emit Address(contractAddress);
    }
}
```

The deployer is partial rewrite of the one at this ethereum [blogpost](https://medium.com/@kalexotsu/writing-evm-logic-in-opcodes-deploying-opcode-logic-on-chain-205618fee38d) showing great examples. And also can't recommend enough [evm codes](https://www.evm.codes/) for analyzing opcode outputs.

If you look closely at the deployer you'll see that our original bytecode is embedded in `mstore` at the end before 0 padding. So the additional bytecode is:

```
0x6008600c60003960086000f3
```

If we analyze the opcodes we get this sequence:

```
[00]	PUSH1	08
[02]	PUSH1	0c
[04]	PUSH1	00
[06]	CODECOPY
[07]	PUSH1	08
[09]	PUSH1	00
[0b]	RETURN
```

00-06 - copies the code of current environment to memory. The size (08) and offset (0c) corresponds to our `0x602a5f5260205ff3` bytecode.
07-0b - return the value from memory. Since we just copied our bytecode to (00) location that is what is going to be returned. Now we have our bytecode deployed to as a new contract.

If you want to confirm that the correct code is stored at the address you can use the "eth_getCode" rpc command.

```json
{
  "jsonrpc": "2.0",
  "method": "eth_getCode",
  "params": ["<contract address>", "latest"],
  "id": 1
}
```

## 3. Submit the new address to the contract

From here on it's just submitting the contract address to the MagicNum contract:

```js
await contract.setSolver('<contract address>')
```
