# Basic IRC2 token

## Initial Requirements

[Install tbears](https://www.icondev.io/docs/tbears-installation)

[Install prerequisties](../../getting-started/prerequisites.md)

## Initalize project

```text
mkdir token-test
cd token-test
```

Then, use `tbears init` command to create a SCORE project.

```text
tbears init sampletoken SampleToken
cd sampletoken
```

Boilerplate for your token will be initialized. The project tree is:

```text
token-test
└── sampletoken
    ├── __init__.py
    ├── package.json
    ├── tests
    │   ├── __init__.py
    │   ├── test_integrate_token.py
    │   └── test_unit_token.py
    └── sampletoken.py
```

Remove _tests_ directory from token as we will not be using it to test our token. Also, the imports in the files of _tests_ directory causes problem while testing with jupyter notebook.

```text
rm -rf tests
```

Now, install [**ODI Contracts**](testbasictoken.md) library. Package import is prohibited except iconservice and the files in your deployed SCORE folder tree. So, the `ODI Contracts` library should be inside the directory, and cannot be imported from outside.

```text
pip install odi-token-contracts -t .
```

`ODIContracts` library will be in sampletoken directory along with **\_**_**init\_**_**.py**, **package.json** and **sampletoken.py**. The tree structure is:

```text
token-test
├── sampletoken
    ├── __init__.py
    ├── ODIContracts
    │   ├── access
    │   │   ├── __init__.py
    │   │   ├── role
    │   │   │   ├── BurnerRole.py
    │   │   │   ├── __init__.py
    │   │   │   ├── MinterRole.py
    │   │   │   ├── PauserRole.py
    │   │   └── roles.py
    │   ├── __init__.py
    │   ├── tokens
    │   │   ├── IIRC2.py
    │   │   ├── __init__.py
    │   │   ├── IRC2burnable.py
    │   │   ├── IRC2capped.py
    │   │   ├── IRC2mintable.py
    │   │   ├── IRC2pausable.py
    │   │   ├── IRC2.py
    │   │   ├── IRC2snapshot.py
    │   └── utils
    │       ├── checks.py
    │       ├── consts.py
    │       ├── __init__.py
    │       ├── pausable.py
    ├── odi_token_contracts-0.0.3.dist-info
    ├── package.json
    └── sampletoken.py
```

{% hint style="info" %}
All `__pycache__` folders and contents of `odi_token_contracts-0.0.1.dist-info` are ignored in this tree.
{% endhint %}

Then, change **sampletoken.py** to this.

```python
from iconservice import *
from .ODIContracts.tokens.IRC2 import IRC2

TAG = 'SampleToken'

class SampleToken(IRC2):
    pass
```

Then, go back to _token-test_ directory.

```text
cd ..
```

You'll need [**repeater**](https://github.com/OpenDevICON/opendevicon/blob/master/repeater.py) module during testing. Run this command to get repeater module in the current directory.

```text
wget https://raw.githubusercontent.com/OpenDevICON/opendevicon/master/repeater.py
```

Now, you have **repeater.py** module in _token-test_ directory.

Now, download this [**notebook**](https://gist.github.com/lilixac/4541fa7f80f7c40a2ac00a795c7b11d5) to test IRC2 tokens. Run this command to get this notebook in the current directory.

```text
https://gist.githubusercontent.com/lilixac/4541fa7f80f7c40a2ac00a795c7b11d5/raw/6a553d9d37bfb7e0926c4e24b1db15fe01a26b8a/token-test.ipynb
```

Now, open jupyter-notebook in the this directory using the following command:

```text
jupyter-notebook
```

Now you can see a Jupyter instance running in your browser.

Click on **token-test.ipnyb** file which will be our main Jupyter Notebook file to test IRC2 tokens.

Then, execute the blocks to **import necessary packages**, **icon service**, **NID**, **Governance Address** and **create new wallet**.

You now should have `deployer_wallet` and `caller_wallet` address. `deployer_wallet` will be used to send IRC2 Token. `caller_wallet` will be used as a random wallet to test methods.

{% hint style="info" %}
Load test ICX to deployer\_wallet address from the [ibriz faucet](https://icon-faucet.ibriz.ai/).
{% endhint %}

### Deploying the contract

```python
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

The **scoreAddress** of your deployed SCORE from the `tx_result` will be saved to the variable `SCORE_ADDRESS` as you will require it while moving forward.

{% hint style="info" %}
You may encounter `timeout` and `gaierror` error. In that case re-execute the block again.
{% endhint %}

### Check the name, symbol, decimals and total supply

```python
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

```python
external_methods = [deployer_wallet.get_address(), caller_wallet.get_address()]
for address in external_methods:
    call = CallBuilder().from_(deployer_wallet.get_address())\
                    .to(SCORE_ADDRESS)\
                    .method('balanceOf')\
                    .params({'account': address})\
                    .build()
    print(icon_service.call(call))
```

_Note the balance of deployer and caller address_

### Transfer of tokens

Now, we transfer the tokens from the deployer to the caller address.

```python
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

