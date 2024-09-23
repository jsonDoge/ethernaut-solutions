# Answer

Goal: Protect `CryptoVault` from being drained out of tokens.

## 1. Double entry point

As the name suggests even though `CryptoVault` function `sweepToken` protects itself from being drained of `underlying` token, the token implementation itself is vulnerable to a workaround. The malicious actor can pass `LegacyToken` address which has `delegate` address field set to `underlying` token address. So if `LegacyToken` would be passed to `CryptoVault` it would not recognize it as the `underlying` token, but `LegacyToken` will cause the same transfer by calling `delegateTransfer` underneath.

## 2. Prevent transfer

`DoubleEntryPoint` (address same as `underlying`) has a forta middleware modifier check which can be used to raise alerts on custom logic based on `msg.data`.

Since we know that we'll be receiving `to`, `value`, `origSender` and function selector we can craft a bot which would raise the flag if the parameters would match the malicious `sweepToken` call. The parameters should match:

`to` - `sweptTokensRecipient` address
`origSender` - `CryptoVault` address
function selector - `delegateTransfer` (not indicated in this task, but in case `fortaNotify` modifier is used in multiple functions)

## 3. Crafting the bot

The bot is written in accordance with previously mentiond requirements. There is another check that can be added and that is `value` - should match `CryptoVault` balance, but for this challenge it wasn't necessary.

```solidity
contract BestBot {
    address public cryptoVault;
    address public cryptoVaultRecipient;
    bytes4 constant DELEGATE_TRANSFER = 0x9cd1a121;

    constructor(address cryptoVault_, address cryptoVaultRecipient_) {
        cryptoVault = cryptoVault_;
        cryptoVaultRecipient = cryptoVaultRecipient_;
    }

    function handleTransaction(address user, bytes calldata msgData) external {
        (address to, , address origSender) = abi.decode(msgData[4:], (address, uint, address)); // to, value, origSender
        bytes4 functionSelector = bytes4(msgData[0:4]);

        if (
            functionSelector == DELEGATE_TRANSFER &&
            origSender == cryptoVault &&
            to == cryptoVaultRecipient
        ) {
            Forta(msg.sender).raiseAlert(user);
        }
    }
}
```

## 4. Submit to Forta contract

After deploying `BestBot` submit the address to `Forta` contract:

```js
await waitTx(fortaContract.setDetectionBot(`<bot address>`))
```

or using `sendTransaction`

```js
await sendTransaction({
  from: player,
  to: `<forta address>`,
  data: '9e927c68000000000000000000000000<bot address>',
})
```
