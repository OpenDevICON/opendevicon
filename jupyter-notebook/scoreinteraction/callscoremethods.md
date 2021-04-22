# Calling SCORE Methods

This section provides an explanation about ways of executing various method types in different Jupyter Notebook cells.

## Readonly Method

First we need to check the status of our game whether it's on or off.

For that we have **get\_game\_status** method in our SCORE.

Execute this cell using Ctrl + Enter in Jupyter Notebook.

```python
_call = CallBuilder()\
    .from_(deployer_wallet.get_address())\
    .to(<YOUR_SCORE_ADDRESS>)\
    .method("get_game_status")\
    .build()
response = icon_service.call(_call)
print(response)
```

{% hint style="info" %}
**CallBuilder\(\)** doesn't send any transaction to the blockchain.
{% endhint %}

If the response is **True**, the game is on else it is turned off.

## External Method

If the game is off you need to change the game status to on before being able to play the game.

For that we have **toggle\_game\_status** method in our SCORE.

Execute this cell using Ctrl + Enter in Jupyter Notebook.

{% hint style="warning" %}
Your game will be turned off if it is already in **"on"** status.
{% endhint %}

```python
call_transaction = CallTransactionBuilder()\
    .from_(deployer_wallet.get_address())\
    .to(<YOUR_SCORE_ADDRESS>)\
    .nid(NID)\
    .nonce(100)\
    .method("toggle_game_status")\
    .build()

estimate_step = icon_service.estimate_step(call_transaction)
step_limit = estimate_step + 100000
# Returns the signed transaction object having a signature
signed_transaction = SignedTransaction(call_transaction, deployer_wallet,step_limit)

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

If the status is **"1"**, your transaction succeeded. You can copy the **tx\_hash** and see your transaction in the [**tracker**](https://bicon.tracker.solidwallet.io/).

Now you can also execute above readonly cell to see change in result.

## Payable Method

Before being able to play the game ,one more thing you need to do is to fill some ICX in Dice SCORE for treasury. For that goto section [**ICX Transfer**](../icxtransfer.md)

Now lets place our first bet using method **call\_bet** in our Dice SCORE.

Execute this cell using Ctrl + Enter in Jupyter Notebook.

```python
params={
    "upper": 50,
    "lower": 5,
    "user_seed": "Lucky",
    "side_bet_amount": 10**18,
    "side_bet_type":"icon_logo1"
}
# Skip side_bet_amount and side_bet_type if you dont want to play sidebet

call_transaction = CallTransactionBuilder()\
    .from_(caller_wallet.get_address())\
    .to(<YOUR_SCORE_ADDRESS>)\
    .nid(NID)\
    .nonce(100)\
    .value(10**18)\
    .method("call_bet")\
    .params(params)\
    .build()

estimate_step = icon_service.estimate_step(call_transaction)
step_limit = estimate_step + 100000
# Returns the signed transaction object having a signature
signed_transaction = SignedTransaction(call_transaction, caller_wallet,step_limit)

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

You can get the information about your win, loss and payout amount from the **eventlogs**.

