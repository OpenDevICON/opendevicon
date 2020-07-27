# Create your own IRC2 token

## Initial Requirements
[Install tbears](https://www.icondev.io/docs/tbears-installation)

[Install prerequisties](prerequisites.md)

## Initalize project
```Shell
$ mkdir token-test
$ cd token-test
``` 

Then, use `tbears init` command to create a SCORE project.

```
$ tbears init sampletoken SampleToken
$ cd sampletoken
```

Boilerplate for your token will be initialized. The project tree is:
```
└── token-test
    └── sampletoken
        ├── __init__.py
        ├── package.json
        ├── tests
        │   ├── __init__.py
        │   ├── test_integrate_token.py
        │   └── test_unit_token.py
        └── sampletoken.py
```   

Remove tests directory from token as we will not be using it to test our token. Also, the imports in the files of tests directory causes problem while testing with jupyter notebook.

```Shell
$ rm -rf tests
```

Now, install **[ODI Contracts]()** library. Package import is prohibited except iconservice and the files in your deployed SCORE folder tree. So, the ODI Contracts library should be inside the directory, and cannot be imported from outside. 

```Shell
$ pip install odi-token-contracts -t .
```

ODIContracts library will be in sampletoken directory along with \__init__.py, package.json and sampletoken.py. The tree structure is:

```
token-test
.
├── sampletoken
    ├── __init__.py
    ├── ODIContracts
    │   ├── access
    │   │   ├── __init__.py
    │   │   ├── role
    │   │   │   ├── BurnerRole.py
    │   │   │   ├── __init__.py
    │   │   │   ├── MinterRole.py
    │   │   │   ├── PauserRole.py
    │   │   └── roles.py
    │   ├── __init__.py
    │   ├── tokens
    │   │   ├── IIRC2.py
    │   │   ├── __init__.py
    │   │   ├── IRC2burnable.py
    │   │   ├── IRC2capped.py
    │   │   ├── IRC2mintable.py
    │   │   ├── IRC2pausable.py
    │   │   ├── IRC2.py
    │   │   ├── IRC2snapshot.py
    │   └── utils
    │       ├── checks.py
    │       ├── consts.py
    │       ├── __init__.py
    │       ├── pausable.py
    ├── odi_token_contracts-0.0.3.dist-info
    ├── package.json
    └── sampletoken.py

```
{% hint style="info"%}
All `__pycache__` folders and contents of `odi_token_contracts-0.0.1.dist-info` are ignored in this tree. 
{% endhint %}

Then, change sampletoken.py to this. 

```Python
from iconservice import *
from .ODIContracts.tokens.IRC2 import IRC2

TAG = 'SampleToken'

class SampleToken(IRC2):
    pass
```
---

Then, go back to token-test directory.
```shell
$ cd ..
```

You'll need [**repeater**](https://raw.githubusercontent.com/OpenDevICON/opendevicon/master/repeater.py) module during testing. Run this command to get repeater module in the current directory.

```shell
$ wget https://raw.githubusercontent.com/OpenDevICON/opendevicon/master/repeater.py
```

Now, you'll have repeater.py module in token-test directory.  \

Now, open jupyter-notebook using the following command:

```shell
$ jupyter-notebook
```

Now you can see a Jupyter instance running in your browser. Click New, and select python 3 in the dropdown to create a new notebook with python3. Then, follow the steps as mentioned in **[SCORE Interaction](https://docs.opendevicon.io/v/development/jupyter-notebook/scoreinteraction#setting-up-environment)** from setting-up-environment.

You now should have deployer_wallet and caller_wallet address. Load test ICX to deployer_wallet address. caller_wallet will be used as a random wallet to test methods.


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
    .content(gen_deploy_data_content('sampletoken'))\
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
The decorator `@retry()` is imported from the **repeater** module.


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

tx_result = get_tx_result(tx_hash)
tx_result
```

Now re-execute the block to check the balance of deployer and caller address. 5 tokens will be transferred to the random address from deployer address. 


CONGRATULATIONS! You have successfully created a token in ICON blockchain, and transferred fund from one address to another address!