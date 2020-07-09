# Creating your own IRC2 token

**First git clone the repo.**

## Make changes in token.py

```Python
from iconservice import *
from .tokens.IRC2 import IRC2

TAG = 'SampleToken'

class SampleToken(IRC2):
    pass
```

**[Install requirements](docs/prerequisites.md))**

## Now open jupyter notebook

**Read [SCORE Interaction](docs/scoreInteraction.md) first**

## Setting up environment
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
    .from_(operator.get_address())\
    .to(GOVERNANCE_ADDRESS)\
    .nid(3)\
    .nonce(100)\
    .content_type("application/zip")\
    .content(gen_deploy_data_content('ODIContracts'))\
    .params(DEPLOY_PARAMS)\
    .build()

estimate_step = icon_service.estimate_step(deploy_transaction)
step_limit = estimate_step + 100000
signed_transaction = SignedTransaction(deploy_transaction, operator, step_limit)
tx_hash = icon_service.send_transaction(signed_transaction)

```
```Python
tx_result = icon_service.get_transaction_result(tx_hash)
tx_result
```
Save the **scoreAddress** of your deployed SCORE from the **tx_result** as you will require it while moving forward.

```Python
SCORE_ADDRESS = "<score_address_obtained_after_deploying_the_contract>"
```

## Updating SCORE
```Python
UPDATE_PARAMS =  {
            "_tokenName": "TestToken2",
            "_symbolName": "TK2",
            "_initialSupply": 1000,
            "_decimals": 18
        }
        
deploy_transaction = DeployTransactionBuilder()\
    .from_(operator.get_address())\
    .to(SCORE_ADDRESS)\
    .nid(3)\
    .nonce(100)\
    .content_type("application/zip")\
    .content(gen_deploy_data_content('ODIContracts'))\
    .params(UPDATE_PARAMS)\
    .build()

estimate_step = icon_service.estimate_step(deploy_transaction)
step_limit = estimate_step + 100000
signed_transaction = SignedTransaction(deploy_transaction, operator, step_limit)
tx_hash = icon_service.send_transaction(signed_transaction)
```
```Python
tx_result = icon_service.get_transaction_result(tx_hash)
tx_result
```

Now as you are ready with your own Dice SCORE deployed in testnet. Let us play some game in the Jupyter Notebook using your SCORE.

### Check the name, symbol, decimals and total supply
```Python
external_methods = ["name", "symbol", "decimals", "totalSupply"]
for method in external_methods:
    call = CallBuilder().from_(operator.get_address())\
                    .to(SCORE_ADDRESS)\
                    .method(method)\
                    .build()
    print(icon_service.call(call))

``` 
### Transfer of tokens
```Python
params={
    "_to": random_wallet.get_address(),
    "_value": 5,
}

call_transaction = CallTransactionBuilder()\
    .from_(operator.get_address())\
    .to(SCORE_ADDRESS)\
    .nid(3)\
    .nonce(100)\
    .method("Transfer")\
    .params(params)\
    .build()

estimate_step = icon_service.estimate_step(call_transaction)
step_limit = estimate_step + 100000
# Returns the signed transaction object having a signature
signed_transaction = SignedTransaction(call_transaction, operator,step_limit)

# Sends the transaction
tx_hash = icon_service.send_transaction(signed_transaction)
tx_hash
```
```Python
tx_result = icon_service.get_transaction_result(tx_hash)
tx_result
```
_Now reexecute the block to check the balance of deployer and random address._

<!-- 
## Now, To implement mintable and burnable

_Change token.py to this to implement burnable and mintable_
```Python
from iconservice import *
from .tokens.IRC2burnable import IRC2Burnable
from .tokens.IRC2mintable import IRC2Mintable

TAG = 'SampleToken'

class SampleToken(IRC2Mintable, IRC2Burnable):
    pass
```

**Reexecute blocks 1 to 8**

### Testing mint method
_Calling mint methods_
```Python
params = {'_value':100000}
mint_transaction = CallTransactionBuilder()\
    .from_(operator.get_address())\
    .to(SCORE_ADDRESS)\
    .nid(3)\
    .nonce(100)\
    .method('mint')\
    .params(params)\
    .build()

estimate_step = icon_service.estimate_step(mint_transaction)
step_limit = estimate_step + 100000
signed_transaction = SignedTransaction(mint_transaction, operator, step_limit)

tx_hash = icon_service.send_transaction(signed_transaction)

@retry(JSONRPCException, tries=10, delay=1, back_off=2)
def get_tx_result(_tx_hash):
    tx_result = icon_service.get_transaction_result(_tx_hash)
    return tx_result

get_tx_result(tx_hash)
```
**Now reexecute blocks to check the total supply and balance again**
100000 tokens are added to total_supply.
100000 tokens are added to the balance of operator address.

### Testing mintTo method

```Python
params = {'_account':RANDOM_ADDRESS,'_value':100000}
mintTo_transaction = CallTransactionBuilder()\
    .from_(operator.get_address())\
    .to(SCORE_ADDRESS)\
    .nid(3)\
    .nonce(100)\
    .method('mint')\
    .params(params)\
    .build()

estimate_step = icon_service.estimate_step(mintTo_transaction)
step_limit = estimate_step + 100000
signed_transaction = SignedTransaction(mintTo_transaction, operator, step_limit)

tx_hash = icon_service.send_transaction(signed_transaction)

@retry(JSONRPCException, tries=10, delay=1, back_off=2)
def get_tx_result(_tx_hash):
    tx_result = icon_service.get_transaction_result(_tx_hash)
    return tx_result

get_tx_result(tx_hash)
```
**Now reexecute blocks to check the total supply and balance of random address again**
100000 tokens are added to total_supply.
100000 tokens is added to the balance of RANDOM_ADDRESS.

### Testing burn method
```Python
params = {'_amount':100000}
burn_transaction = CallTransactionBuilder()\
    .from_(operator.get_address())\
    .to(SCORE_ADDRESS)\
    .nid(3)\
    .nonce(100)\
    .method('burn')\
    .params(params)\
    .build()

estimate_step = icon_service.estimate_step(burn_transaction)
step_limit = estimate_step + 100000
signed_transaction = SignedTransaction(burn_transaction, operator, step_limit)

tx_hash = icon_service.send_transaction(signed_transaction)

@retry(JSONRPCException, tries=10, delay=1, back_off=2)
def get_tx_result(_tx_hash):
    tx_result = icon_service.get_transaction_result(_tx_hash)
    return tx_result

get_tx_result(tx_hash)
```
**Now reexecute blocks to check the total supply and balance again**
100000 tokens are destroyed from the total_supply.
100000 tokens are destroyed from the operator address.

###Testing burnFrom method
```Python
params = {'_account':RANDOM_ADDRESS,'_amount':100000}
burnFrom_transaction = CallTransactionBuilder()\
    .from_(operator.get_address())\
    .to(SCORE_ADDRESS)\
    .nid(3)\
    .nonce(100)\
    .method('burnFrom')\
    .params(params)\
    .build()

estimate_step = icon_service.estimate_step(burnFrom_transaction)
step_limit = estimate_step + 100000
signed_transaction = SignedTransaction(burnFrom_transaction, operator, step_limit)

tx_hash = icon_service.send_transaction(signed_transaction)

@retry(JSONRPCException, tries=10, delay=1, back_off=2)
def get_tx_result(_tx_hash):
    tx_result = icon_service.get_transaction_result(_tx_hash)
    return tx_result

get_tx_result(tx_hash)
```
**Now reexecute blocks to check the total supply and balance of random address again**
100000 tokens are destroyed from the total_supply.
100000 tokens are destroyed from the balance of RANDOM_ADDRESS.




 -->