# Create your own IRC2 token

**First, [Install requirements](prerequisties.md)**<br>

## Cloning the project
You can get the project repo from [**Github**]()

Run command: 
```Shell
$ git clone 
```

# After initialization...

```Shell
touch token.py
```
Add this to token.py

```Python
from iconservice import *
from .tokens.IRC2 import IRC2

TAG = 'SampleToken'

class SampleToken(IRC2):
    pass
```

Now, read **[SCORE Interaction](scoreinteraction.md)**. 
Make a new python notebook, and follow the steps as mentioned in **SCORE Interaction**. Now you should have deployer_wallet and caller_wallet address. Load funds to deployer_wallet address. caller_wallet will be used as a random wallet to test methods.


### Deploying the contract

```Python
DEPLOY_PARAMS =  {
            "_tokenName": "TestToken",
            "_symbolName": "TK",
            "_initialSupply": 1000,
            "_decimals": 18
        }


deploy_transaction = DeployTransactionBuilder()\
    .from_(deployer_wallet.get_address())\
    .to("cx0000000000000000000000000000000000000000")\
    .nid(NID)\
    .nonce(100)\
    .content_type("application/zip")\
    .content(gen_deploy_data_content('ODIContracts'))\
    .params(DEPLOY_PARAMS)\
    .build()

estimate_step = icon_service.estimate_step(deploy_transaction)
step_limit = estimate_step + 100000
signed_transaction = SignedTransaction(deploy_transaction, deployer_wallet, step_limit)

tx_hash = icon_service.send_transaction(signed_transaction)

@retry(JSONRPCException, tries=10, delay=1, back_off=2)
def get_tx_result(_tx_hash):
    tx_result = icon_service.get_transaction_result(_tx_hash)
    return tx_result

tx_result = get_tx_result(tx_hash)
SCORE_ADDRESS = tx_result['scoreAddress']
```

The **scoreAddress** of your deployed SCORE from the **tx_result** will be saved to the variable SCORE_ADDRESS as you will require it while moving forward.

### Check the name, symbol, decimals and total supply
```Python
external_methods = ["name", "symbol", "decimals", "totalSupply"]
for method in external_methods:
    call = CallBuilder().from_(deployer_wallet.get_address())\
                    .to(SCORE_ADDRESS)\
                    .method(method)\
                    .build()
    print(icon_service.call(call))
``` 
You have successfully created a token!

### Check the total number of tokens of caller and deployer address
```Python
external_methods = [deployer_wallet.get_address(), caller_wallet.get_address()]
for address in external_methods:
    call = CallBuilder().from_(deployer_wallet.get_address())\
                    .to(SCORE_ADDRESS)\
                    .method('balanceOf')\
                    .params({'account': address})\
                    .build()
    print(icon_service.call(call))
```
*Note the balance of deployer and caller address*

### Transfer of tokens
Now, we transfer the tokens from the deployer to the caller address.
```Python
params={
    "_to": caller_wallet.get_address(),
    "_value": 5,
}

call_transaction = CallTransactionBuilder()\
    .from_(deployer_wallet.get_address())\
    .to(SCORE_ADDRESS)\
    .nid(3)\
    .nonce(100)\
    .method("transfer")\
    .params(params)\
    .build()


estimate_step = icon_service.estimate_step(call_transaction)
step_limit = estimate_step + 100000
signed_transaction = SignedTransaction(call_transaction, deployer_wallet, step_limit)

tx_hash = icon_service.send_transaction(signed_transaction)

@retry(JSONRPCException, tries=10, delay=1, back_off=2)
def get_tx_result(_tx_hash):
    tx_result = icon_service.get_transaction_result(_tx_hash)
    return tx_result

get_tx_result(tx_hash)
```

Now reexecute the block to check the balance of deployer and caller address. 5 tokens will be transferred to the random address from deployer address. 
<br>
You have successfully created a token in ICON blockchain, and transferred fund from to another address!