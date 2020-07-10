## Create your own IRC2 token

**First git clone the repo.**

### Make changes in token.py

```Python
from iconservice import *
from .tokens.IRC2 import IRC2

TAG = 'SampleToken'

class SampleToken(IRC2):
    pass
```

**[Install requirements]('https://ibriz.gitbook.io/icondev-1/-M8GZOLeT06JW0Jng2Pn/getting-started/prerequisites',"Prerequisties")**

### Now open jupyter notebook

**Read [SCORE Interaction]('https://ibriz.gitbook.io/icondev-1/-M8GZOLeT06JW0Jng2Pn/jupyter-notebook/scoreinteraction') first**

### Setting up environment
Before we move on to using the SDK we first need to initialize our environment.
### Importing packages
All the necessary packages required for using SDK must be imported.
```py
from iconsdk.exception import JSONRPCException
from iconsdk.libs.in_memory_zip import gen_deploy_data_content
from iconsdk.icon_service import IconService
from iconsdk.providers.http_provider import HTTPProvider
from iconsdk.builder.transaction_builder import CallTransactionBuilder, TransactionBuilder, DeployTransactionBuilder
from iconsdk.builder.call_builder import CallBuilder
from iconsdk.signed_transaction import SignedTransaction
from iconsdk.wallet.wallet import KeyWallet
from iconsdk.utils.convert_type import convert_hex_str_to_int
from repeater import retry
import requests
import json
import pickle
import csv
import os
```
{% hint style="info"%}
For error msg like `package not found`

Run command:-
```py
$ pip install <package-name>
```
{% endhint %}

### Setting up URL
Create an **IconService** instance and set a provider. The **HTTPProvider** takes one of the following URL where the server can be found. For this documentation we will be using [**Yeouido**](https://bicon.tracker.solidwallet.io/) tesnet. 
```py
# Mainnet
# icon_service = IconService(HTTPProvider("https://ctz.solidwallet.io", 3))
# Euljiro
# icon_service = IconService(HTTPProvider("https://test-ctz.solidwallet.io",3))
# Yeouido
icon_service = IconService(HTTPProvider("https://bicon.net.solidwallet.io", 3))
# Pagoda
# icon_service = IconService(HTTPProvider("https://zicon.net.solidwallet.io", 3))
# Custom
# icon_service = IconService(HTTPProvider("http://54.186.60.134:9000/", 3))
```
### Setting up NID
Netword ID for Yeoido tesnet is 3.
```py
# Mainnet
# NID = 1
# Euljiro Testnet
# NID = 2
# Yeouido Testnet // Tbears
NID = 3
# Pagoda Testnet
# NID = 80
```

### Creating wallets
We will create two wallets, one for deploying SCORE and other for transferring tokens. 
```py
deployer_wallet = KeyWallet.create() 
print("address: ", deployer_wallet.get_address())
random_wallet = KeyWallet.create() 
print("address: ", random_wallet.get_address())
```

{% hint style="info" %}
Go to [**ibriz-faucet**](https://icon-faucet.ibriz.ai/) to receive test ICXs in deployer_wallet.
{% endhint %}

Now we are all set to use the SDK in Jupyter Notebook. You can get the official documentation of PythonSDK for ICON [**here**](https://www.icondev.io/docs/python-sdk). 

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

get_tx_result(tx_hash)
```
Save the **scoreAddress** of your deployed SCORE from the **tx_result** as you will require it while moving forward.

```Python
SCORE_ADDRESS = "<score_address_obtained_after_deploying_the_contract>"
```

### Updating SCORE
```Python
UPDATE_PARAMS =  {
            "_tokenName": "TestToken1",
            "_symbolName": "TK1",
            "_initialSupply": 1100,
            "_decimals": 18
        }


update_transaction = DeployTransactionBuilder()\
    .from_(deployer_wallet.get_address())\
    .to(SCORE_ADDRESS)\
    .nid(NID)\
    .nonce(100)\
    .content_type("application/zip")\
    .content(gen_deploy_data_content('ODIContracts'))\
    .params(UPDATE_PARAMS)\
    .build()

estimate_step = icon_service.estimate_step(update_transaction)
step_limit = estimate_step + 100000
signed_transaction = SignedTransaction(update_transaction, deployer_wallet, step_limit)

tx_hash = icon_service.send_transaction(signed_transaction)

@retry(JSONRPCException, tries=10, delay=1, back_off=2)
def get_tx_result(_tx_hash):
    tx_result = icon_service.get_transaction_result(_tx_hash)
    return tx_result

get_tx_result(tx_hash)
```

Now as you are ready with your own token deployed in testnet. Let us test transfer method in the Jupyter Notebook using your SCORE.

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

### Check the total number of tokens of random and deployer address
```Python
external_methods = [deployer_wallet.get_address(), random_wallet.get_address()]
for address in external_methods:
    call = CallBuilder().from_(deployer_wallet.get_address())\
                    .to(SCORE_ADDRESS)\
                    .method('balanceOf')\
                    .params({'account': address})\
                    .build()
    print(icon_service.call(call))
```
*Note the balance of deployer and random address*

### Transfer of tokens
Now, we transfer the tokens from the deployer to the ransom address.
```Python
params={
    "_to": random_wallet.get_address(),
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
Now reexecute the block to check the balance of deployer and random address. 5 tokens will be transferred to the random address from deployer address.

## Implement mintable and burnable tokens

Change token.py to implement IRC2burnable and IRC2mintable.<br>
IRC2 Mintable and IRC2 Burnable inherits from IRC2. 

```Python
from iconservice import *
from .tokens.IRC2burnable import IRC2Burnable
from .tokens.IRC2mintable import IRC2Mintable

TAG = 'SampleToken'

class SampleToken(IRC2Pausable, IRC2Mintable):
    pass
```

Now, since we changed the token, we need to update the contract. Execute the block to update the contract again without making change to other blocks.

### Testing minting of tokens
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

@retry(JSONRPCException, tries=10, delay=1, back_off=2)
def get_tx_result(_tx_hash):
    tx_result = icon_service.get_transaction_result(_tx_hash)
    return tx_result

get_tx_result(tx_hash)
```
After this block finishes executing, now reexecute the block to check the total supply and deployer balance. 5 tokens are added to total supply as well as the deployer address. 

### Testing mintTo method

Using this method, tokens can be minted to other address as well. However, only the deployer can call this method.
```Python
params={
    "_account": random_wallet.get_address(),
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

@retry(JSONRPCException, tries=10, delay=1, back_off=2)
def get_tx_result(_tx_hash):
    tx_result = icon_service.get_transaction_result(_tx_hash)
    return tx_result

get_tx_result(tx_hash)
```
After this block finishes executing, now reexecute the block to check the total supply and random address balance. 5 tokens are added to total supply as well as the random address.  

### Testing burning of tokens

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

@retry(JSONRPCException, tries=10, delay=1, back_off=2)
def get_tx_result(_tx_hash):
    tx_result = icon_service.get_transaction_result(_tx_hash)
    return tx_result

get_tx_result(tx_hash)
```
After this block finishes executing, now reexecute the block to check the total supply and deployer balance. 5 tokens is subtracted from total supply as well as from the deployer address. 

### Testing burnFrom method
Using this method, tokens can be destroyed from other address as well. However, only the deployer can call this method.
```Python
params={
    "_account": random_wallet.get_address(),
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

@retry(JSONRPCException, tries=10, delay=1, back_off=2)
def get_tx_result(_tx_hash):
    tx_result = icon_service.get_transaction_result(_tx_hash)
    return tx_result

get_tx_result(tx_hash)
```

After this block finishes executing, now reexecute the block to check the total supply and random address balance. 5 tokens is deducted from total supply as well as from the random address. 