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

**[Install requirements]((docs/prerequisites.md))**

## Now open jupyter notebook

**Read [SCORE Interaction](docs/scoreInteraction.md) first**

#### Import all the necessary packages.:

```Python
#Import necessary packages
from iconsdk.icon_service import IconService
from iconsdk.providers.http_provider import HTTPProvider
from iconsdk.builder.transaction_builder import CallTransactionBuilder, TransactionBuilder,DeployTransactionBuilder,MessageTransactionBuilder
from iconsdk.builder.call_builder import CallBuilder
from iconsdk.signed_transaction import SignedTransaction
from iconsdk.wallet.wallet import KeyWallet
from iconsdk.utils.convert_type import convert_hex_str_to_int
import requests
import json
import pickle
import csv
import os
from repeater import retry
from iconsdk.libs.in_memory_zip import gen_deploy_data_content
```
### [1] List required address.

```Python
GOVERNANCE_ADDRESS = "cx0000000000000000000000000000000000000000"
SCORE_ADDRESS = "cxf145f42657dbf2f98f02427b55e27c67d2951220"
RANDOM_ADDRESS = "hx43e1f6be06a62a293d06867adc27e7acc9affe69"
DEPLOY_PARAMS = {}
```

### [2] Connect to test net.

```Python
# Yeouido
NID = 3
icon_service = IconService(HTTPProvider("https://bicon.net.solidwallet.io", NID))
```

### [3] Import necessary wallet or create new wallet
*We'll be creating new wallet here*

```Python
operator = KeyWallet.create() 
print("address: ", operator.get_address())
```
{% hint style="info" %}
Go to [**ibriz-faucet**](https://icon-faucet.ibriz.ai/) to receive test ICXs in your wallets.
{% endhint %}

### [4] Deploying the contract

```Python
deploy_params =  {
            "_tokenName": "TestToken",
            "_symbolName": "TTK",
            "_initialSupply": 1000,
            "_decimals": 18,
            # "_cap": 372427 => _cap is set to the highest possible value by default.
        }

deploy_transaction = DeployTransactionBuilder()\
    .from_(operator.get_address())\
    .to("cx0000000000000000000000000000000000000000")\
    .nid(NID)\
    .nonce(100)\
    .content_type("application/zip")\
    .content(gen_deploy_data_content('ODIContracts'))\
    .params(deploy_params)\
    .build()

estimate_step = icon_service.estimate_step(deploy_transaction)
step_limit = estimate_step + 100000
signed_transaction = SignedTransaction(deploy_transaction, operator, step_limit)

tx_hash = icon_service.send_transaction(signed_transaction)

tx_result = icon_service.get_transaction_result(tx_hash)
tx_result
```

_copy the score address and paste it up in Block 2. Execute the second block once again._

### [5] Updating Contract

```Python
update_params =  {
            "_tokenName": "SampleToken",
            "_symbolName": "STK",
            "_initialSupply": 1000,
            "_decimals": 18,
            # "_cap": 2500 # cap is set to the highest possible value by default.
        }

update_transaction = DeployTransactionBuilder()\
    .from_(operator.get_address())\
    .to(<YOUR_SCORE_ADDRESS>)\
    .nid(NID)\
    .nonce(100)\
    .content_type("application/zip")\
    .content(gen_deploy_data_content('ODIContracts'))\
    .params(update_params)\
    .build()

estimate_step = icon_service.estimate_step(update_transaction)
step_limit = estimate_step + 100000
signed_transaction = SignedTransaction(update_transaction, operator, step_limit)

tx_hash = icon_service.send_transaction(signed_transaction)

tx_result = icon_service.get_transaction_result(tx_hash)
return tx_result

```

### [6] Call External ReadOnly Methods

```Python
external_methods = ["name", "symbol", "decimals", "totalSupply"]
for method in external_methods:
    call = CallBuilder().from_(operator.get_address())\
                    .to(SCORE_ADDRESS)\
                    .method(method)\
                    .build()
    print(icon_service.call(call))
```
### A NOTE ON DECIMALS
initial_supply = 2000<br>
It means no. of initial tokens = 2000 * 10 ^ 18

### [7] Check the balance
_Check the balance of the operator and random address_
```Python
addresses = [operator.get_address(), RANDOM_ADDRESS]
for address in addresses:
    call = CallBuilder().from_(operator.get_address())\
                    .to(SCORE_ADDRESS)\
                    .method('balanceOf')\
                    .params({'account': address})\
                    .build()
    print(icon_service.call(call))
```


### [8] Testing Transfer Method

Look at the external transfer method in the code, line 215 {IRC2.py}

```Python
params = {'_to':RANDOM_ADDRESS, '_value':100}
transfer_transaction = CallTransactionBuilder()\
    .from_(operator.get_address())\
    .to(SCORE_ADDRESS)\
    .nid(3)\
    .nonce(100)\
    .method('transfer')\
    .params(params)\
    .build()

estimate_step = icon_service.estimate_step(transfer_transaction)
step_limit = estimate_step + 100000
signed_transaction = SignedTransaction(transfer_transaction, operator, step_limit)

tx_hash = icon_service.send_transaction(signed_transaction)

@retry(JSONRPCException, tries=10, delay=1, back_off=2)
def get_tx_result(_tx_hash):
    tx_result = icon_service.get_transaction_result(_tx_hash)
    return tx_result

get_tx_result(tx_hash)

```

**Now reexecute block 7 again**
100 tokens will be transferred from operator to RANDOM_ADDRESS.


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

**Reexecute blocks 1 to 7**

### [9] Testing mint method
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
**Now reexecute blocks 6 and 7 again**
100000 tokens are added to total_supply.
100000 tokens are added to the balance of operator address.

### [10] Testing mintTo method

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
**Now reexecute blocks 6 and 7 again**
100000 tokens are added to total_supply.
100000 tokens is added to the balance of RANDOM_ADDRESS.

### [11] Testing burn method
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
**Now reexecute blocks 6 and 7 again**
100000 tokens are destroyed from the total_supply.
100000 tokens are destroyed from the operator address.

### [12] Testing burnFrom method
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
**Now reexecute blocks 6 and 7 again**
100000 tokens are destroyed from the total_supply.
100000 tokens are destroyed from the balance of RANDOM_ADDRESS.




