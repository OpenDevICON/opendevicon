# Creating your own IRC2 token

-> First git clone the repo.

## Make changes in token.py

```Python
from iconservice import *
from .tokens.IRC2 import IRC2

TAG = 'SampleToken'

class SampleToken(IRC2):
	pass
```

-> Install requirements (Look at prerequisties)

## Now open jupyter notebook

READ SCORE DEPLOYMENT SECTION

#### Block 1:

```Python
#Import necessary packages
from iconsdk.icon_service import IconService
from iconsdk.providers.http_provider import HTTPProvider
from iconsdk.builder.transaction_builder import CallTransactionBuilder, TransactionBuilder,DeployTransactionBuilder,MessageTransactionBuilder
from iconsdk.builder.call_builder import CallBuilder
from iconsdk.signed_transaction import SignedTransaction
from iconsdk.wallet.wallet import KeyWallet
from iconsdk.utils.convert_type import convert_hex_str_to_int
from iconsdk.libs.in_memory_zip import gen_deploy_data_content
import requestsNow, to implement the mintable and burnable
import json
import pickle
import csv
import os
from repeater import retry
```

### Block 2

```Python
GOVERNANCE_ADDRESS = "cx0000000000000000000000000000000000000000"
SCORE_ADDRESS = "cxf145f42657dbf2f98f02427b55e27c67d2951220"
RANDOM_ADDRESS = "hx43e1f6be06a62a293d06867adc27e7acc9affe69"
DEPLOY_PARAMS = {}
```

### Block 3

Select test net, NID

### Block 4

Load funds in the wallet

### Block 5

```Python
deploy_params =  {
            "_tokenName": "TestToken",
            "_symbolName": "TK",
						"_initialSupply": 1000,
            "_decimals": 18,
            # "_cap": 372427 cap is set to the highest possible value by default.
		}

deploy_transaction = DeployTransactionBuilder()\
    .from_(deployer_wallet.get_address())\
    .to("cx0000000000000000000000000000000000000000")\
    .nid(NID)\
    .nonce(100)\
    .content_type("application/zip")\
    .content(gen_deploy_data_content('ODIContracts'))\
    .params(deploy_params)\
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

-> copy the score address and paste it up in Block 2. Execute the second block once again.

### Block 6

```Python
update_params =  {
            "_tokenName": "SampleToken",
            "_symbolName": "STK",
						"_initialSupply": 2000,
            "_decimals": 18,
            "_cap": 2500 # cap is set to the highest possible value by default.
		}

update_transaction = DeployTransactionBuilder()\
    .from_(deployer_wallet.get_address())\
    .to(<YOUR_SCORE_ADDRESS>)\
    .nid(NID)\
    .nonce(100)\
    .content_type("application/zip")\
    .content(gen_deploy_data_content('ODIContracts'))\
    .params(update_params)\
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

### Block 7

_Calling the external readonly methods_

```Python
external_methods = ["name", "symbol", "decimals", "totalSupply"]
for method in external_methods:
    call = CallBuilder().from_(operator.get_address())\
                    .to(SCORE_ADDRESS)\
                    .method(method)\
                    .build()
    print(icon_service.call(call))
```
## A NOTE ON DECIMALS
initial_supply = 2000
it means no. of initial tokens = 2000 * 10 ^ 18

### Block 8
_Check the balance of the operator address_
```Python
call = CallBuilder().from_(operator.get_address())\
                    .to(SCORE_ADDRESS)\
                    .method('balanceOf')\
                    .params({'account': operator.get_address()})\
                    .build()
print(icon_service.call(call))
```

### Block 9
_Check the balance of the random address_
```Python
call = CallBuilder().from_(operator.get_address())\
                    .to(SCORE_ADDRESS)\
                    .method('balanceOf')\
                    .params({'account': RANDOM_ADDRESS})\
                    .build()
print(icon_service.call(call))
```

### Block 10

_Calling transfer methods_
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

**Now reexecute blocks 8 and 9 again**
100 tokens will be transferred from operator to RANDOM_ADDRESS.
