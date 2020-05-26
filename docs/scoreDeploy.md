# SCORE Deployment

Our Dice SCORE is ready and we are going to deploy it in our testnet. Now compress the dice folder into **dice.zip**.

 We will use **DeployTransactionBuilder()** to generate an instance of transaction for deploying the SCORE.

{% hint style="warning" %}
Make sure you delete any tests folder inside your main project before compressing!
{% endhint %}

Execute the cell using Ctrl + Enter in Jupyter Notebook and your SCORE will be deployed in the testnet.
```py
deploy_transaction = DeployTransactionBuilder()\
    .from_(deployer_wallet.get_address())\
    .to("cx0000000000000000000000000000000000000000")\
    .nid(NID)\
    .nonce(100)\
    .content_type("application/zip")\
    .content(gen_deploy_data_content('dice.zip'))\
    .build()

estimate_step = icon_service.estimate_step(deploy_transaction)
step_limit = estimate_step + 100000
signed_transaction = SignedTransaction(deploy_transaction, deployer_wallet, step_limit)

tx_hash = icon_service.send_transaction(signed_transaction)

@retry(JSONRPCException, tries=10, delay=1, back_off=2)
def get_tx_result(_tx_hash):
    tx_result = icon_service.get_transaction_result(_tx_hash)
    return tx_result

get_tx_result(tx_hash)
```
Save the **scoreAddress** of your deployed SCORE from the **tx_result** as you will require it while moving forward.


## Updating SCORE
One of the main feature in ICON blockchain is that you can have your updated SCORE in the same SCORE address. Only you have to do is to set the **"to"** param to previous SCORE address rather than Governance address.

Make any change in the dice score and update the dice.zip file.

Execute the cell using Ctrl + Enter in Jupyter Notebook and your SCORE will be updated in the testnet.

```py
update_transaction = DeployTransactionBuilder()\
    .from_(deployer_wallet.get_address())\
    .to(<YOUR_SCORE_ADDRESS>)\
    .nid(NID)\
    .nonce(100)\
    .content_type("application/zip")\
    .content(gen_deploy_data_content('scorezip.zip'))\
    .build()

estimate_step = icon_service.estimate_step(deploy_transaction)
step_limit = estimate_step + 100000
signed_transaction = SignedTransaction(deploy_transaction, deployer_wallet, step_limit)

tx_hash = icon_service.send_transaction(signed_transaction)

@retry(JSONRPCException, tries=10, delay=1, back_off=2)
def get_tx_result(_tx_hash):
    tx_result = icon_service.get_transaction_result(_tx_hash)
    return tx_result

get_tx_result(tx_hash)
```


Now as you are ready with your own Dice SCORE deployed in testnet. Let us play some game in the Jupyter Notebook using your SCORE.