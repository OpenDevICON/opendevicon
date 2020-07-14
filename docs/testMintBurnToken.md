# Implement mintable and burnable tokens

To implement mintable and burnable token, change token.py to implement IRC2burnable and IRC2mintable.<br>

```Python
from iconservice import *
from .ODIContracts.tokens.IRC2burnable import IRC2Burnable
from .ODIContracts.tokens.IRC2mintable import IRC2Mintable

TAG = 'SampleToken'

class SampleToken(IRC2Burnable, IRC2Mintable):
    pass
```

Now, since we changed the token, we need to update the contract. Execute this block to update the contract again without making change to other blocks.

> While updating the contract, the parameters can be changed as well.

```Python
UPDATE_PARAMS =  {
            "_tokenName": "TestToken",
            "_symbolName": "TK",
            "_initialSupply": 1000,
            "_decimals": 18
        }

update_transaction = DeployTransactionBuilder()\
    .from_(deployer_wallet.get_address())\
    .to(SCORE_ADDRESS)\
    .nid(NID)\
    .nonce(100)\
    .content_type("application/zip")\
    .content(gen_deploy_data_content('sampletoken'))\
    .params(UPDATE_PARAMS)\
    .build()

estimate_step = icon_service.estimate_step(update_transaction)
step_limit = estimate_step + 100000
signed_transaction = SignedTransaction(update_transaction, deployer_wallet, step_limit)

tx_hash = icon_service.send_transaction(signed_transaction)
tx_result = icon_service.get_transaction_result(_tx_hash)
tx_result
```


### Test mint method
Using this method, `_amount` tokens can be minted to deployer address. Only the deployer can call this method.
```Python
params={
    "_amount": 5,
}

call_transaction = CallTransactionBuilder()\
    .from_(deployer_wallet.get_address())\
    .to(SCORE_ADDRESS)\
    .nid(3)\
    .nonce(100)\
    .method("mint")\
    .params(params)\
    .build()


estimate_step = icon_service.estimate_step(call_transaction)
step_limit = estimate_step + 100000
signed_transaction = SignedTransaction(call_transaction, deployer_wallet, step_limit)

tx_hash = icon_service.send_transaction(signed_transaction)
tx_result = icon_service.get_transaction_result(_tx_hash)
tx_result
```
After this block finishes executing, reexecute the block to check the total supply and deployer balance. 5 tokens must be added to total supply as well as the deployer address. 

### Test mintTo method

Using this method, tokens can be minted to other address as well. However, only the deployer can call this method. Here, `_amount` number of tokens will be minted to `_account`.
```Python
params={
    "_account": caller_wallet.get_address(),
    "_amount": 5,
}

call_transaction = CallTransactionBuilder()\
    .from_(deployer_wallet.get_address())\
    .to(SCORE_ADDRESS)\
    .nid(3)\
    .nonce(100)\
    .method("mintTo")\
    .params(params)\
    .build()


estimate_step = icon_service.estimate_step(call_transaction)
step_limit = estimate_step + 100000
signed_transaction = SignedTransaction(call_transaction, deployer_wallet, step_limit)

tx_hash = icon_service.send_transaction(signed_transaction)
tx_result = icon_service.get_transaction_result(_tx_hash)
tx_result
```
After this block finishes executing, now reexecute the block to check the total supply and random address balance. 5 tokens are added to total supply as well as the caller address.  

### Test burn method
Using this method, `_amount` tokens can be destroyed from deployer address. Only the deployer can call this method.
```Python
params={
    "_amount": 5,
}

call_transaction = CallTransactionBuilder()\
    .from_(deployer_wallet.get_address())\
    .to(SCORE_ADDRESS)\
    .nid(3)\
    .nonce(100)\
    .method("burn")\
    .params(params)\
    .build()


estimate_step = icon_service.estimate_step(call_transaction)
step_limit = estimate_step + 100000
signed_transaction = SignedTransaction(call_transaction, deployer_wallet, step_limit)

tx_hash = icon_service.send_transaction(signed_transaction)
tx_result = icon_service.get_transaction_result(_tx_hash)
tx_result
```
After this block finishes executing, now reexecute the block to check the total supply and deployer balance. 5 tokens is subtracted from total supply as well as from the deployer address. 

### Test burnFrom method
Using this method, tokens can be destroyed from other address as well. However, only the deployer can call this method.
```Python
params={
    "_account": caller_wallet.get_address(),
    "_amount": 5,
}

call_transaction = CallTransactionBuilder()\
    .from_(deployer_wallet.get_address())\
    .to(SCORE_ADDRESS)\
    .nid(3)\
    .nonce(100)\
    .method("burnFrom")\
    .params(params)\
    .build()


estimate_step = icon_service.estimate_step(call_transaction)
step_limit = estimate_step + 100000
signed_transaction = SignedTransaction(call_transaction, deployer_wallet, step_limit)

tx_hash = icon_service.send_transaction(signed_transaction)
tx_result = icon_service.get_transaction_result(_tx_hash)
tx_result
```

After this block finishes executing, now reexecute the block to check the total supply and random address balance. 5 tokens is deducted from total supply as well as from the caller address. 
