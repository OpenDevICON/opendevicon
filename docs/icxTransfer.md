# ICX Transfer

For transfering a ICX to a particular wallet address or contract address we will make a transaction using **TransactionBuilder()**

Here we will be using this to transfer 99 ICX from our deployer wallet to our dice SCORE.
{% hint style="warning" %}
If you dnt have enough ICX in your wallet then the transfer will fail. Use [**ibriz-faucet**](https://icon-faucet.ibriz.ai/) for getting ICX.

Also use deployer wallet only to make the transfer because fallback method of Dice SCORE only accepts incoming plain ICX from its owner
{% endhint %}

Execute the cell using Ctrl + Enter to transfer ICX to Dice SCORE.
```py
transaction = TransactionBuilder()\
    .from_(deployer_wallet.get_address())\
    .to(<YOUR_SCORE_ADDRESS>)\
    .nid(NID)\
    .nonce(100)\
    .value(50*10**18)\
    .build()

estimate_step = icon_service.estimate_step(transaction)
step_limit = estimate_step + 100000
# Returns the signed transaction object having a signature
signed_transaction = SignedTransaction(transaction, caller_wallet,step_limit)

# Reads params to transfer to nodes
print(signed_transaction.signed_transaction_dict)

# Sends the transaction
tx_hash = icon_service.send_transaction(signed_transaction)

@retry(JSONRPCException, tries=10, delay=1, back_off=2)
def get_tx_result(_tx_hash):
    tx_result = icon_service.get_transaction_result(_tx_hash)
    return tx_result

get_tx_result(tx_hash)
```
Execute the cell to get balance of Dice SCORE after transfer
```py
balance = icon_service.get_balance(<YOUR_SCORE_ADDRESS>)
print(balance)
```
