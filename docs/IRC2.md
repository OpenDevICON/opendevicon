#IRC2 

## Methods

####name
Returns the name of the token. eg. `SampleToken`
```Python
@external(readonly=True)
def name(self) -> str:
```
####symbol
Returns the symbol of the token. eg. `ST`
```Python
@external(readonly=True)
def symbol(self) -> str:
```

####decimals
Returns the number of decimals in the token. It is 18 by default. eg. `18`
```Python
@external(readonly=True)
def decimals(self) -> str:
```

####totalSupply
Returns the total number of tokens in existence.
```Python
@external(readonly=True)
def totalSupply(self) -> int:
```
####balanceOf
Returns the amount of tokens owned by the account with address `account`.
```Python
@external
def balanceOf(self,account: Address) -> int:
```

####transfer
Transfers `_value` amount of tokens to the address `_to` from `self.msg.sender` and fires Transfer eventlog. `self.msg.sender` should have enough balance to transfer to `_to`. If `_to` is a contract, this function must invoke the function `tokenFallback(Address, int, bytes)` in `_to`. If the `tokenFallback` function is not implemented in `_to` (receiver contract), then the transaction must fail and the transfer of tokens should not occur. If `_to` is an externally owned address, then the transaction must be sent without trying to execute `tokenFallback` in `_to`.  `_data` can be attached to this token transaction. `_data` can be empty.
```Python
@external
def transfer(self, _to: Address, _value: int, _data: bytes = None) -> bool:
```

####\_allowance
Returns the number of tokens that the `spender` will be allowed to spend on behalf of `owner`.
```Python
@external
def _allowance(self, owner: Address, spender: Address) -> int:
```

####approve
Sets the allowance value given by the `owner` to the `spender`. Returns a boolean value to check if the operation was successful.
```Python
@external
def approve(self, spender: Address, amount: int) -> bool:
```

####increaseAllowance
Increases the allowance granted to `spender` by `value` amount. Returns a boolean value if the operation was successful.
```Python
@external
def increaseAllowance(self, spender: Address, value: int) -> bool:
```

####decreaseAllowance
Decreases the allowance granted to `spender` by `value` amount. Returns a boolean value if the operation was successful.
```Python
@external
def decreaseAllowance(self, spender: Address, value: int) -> bool:
```

## Eventlogs

#### Transfer
Must trigger on any successful token transfers.
```python
@eventlog(indexed=3)
def Transfer(self, _from: Address, _to: Address, _value: int, _data: bytes):
    pass
```
#### Mint
Triggered after any successful mint and mintTo methods.
```Python
@eventlog(indexed=1)
def Mint(self, account:Address, amount: int):
	pass
```

#### Burn
Triggered after any successful burn and burnFrom methods.
```Python
@eventlog(indexed=1)
def Burn(self, account: Address, amount: int):
	pass
```


#### Token Fallback
A function for handling token transfers, which is called from the token contract, when a token holder sends tokens. `_from` is the address of the sender of the token, `_value` is the amount of incoming tokens, and `_data` is arbitrary attached data. It works by analogy with the fallback function of the normal transactions and returns nothing.
```python
@external
def tokenFallback(self, _from: Address, _value: int, _data: bytes):
```

## Internal Functions

####\_transfer
Transfers `_value` amount from address `_from` to address `_to`. `_data` is optional.
```Python
def _transfer(self, _from: Address, _to: Address, _value: int, _data: bytes = None) -> None:
```

####\_mint
Creates `amount` number of tokens, and assigns to `account`. Increases the balance of that `account` and total supply. Only the minters and address which owns the contract can `_mint` tokens.
```Python
@only_owner
def _mint(self, account:Address, amount:int) -> None:
```

####\_burn
Destroys `amount` number of tokens from account with address `account`. Decreases the balance of that `account` and total supply. Only the burners and address which owns the contract can `_burn` tokens.
```Python
@only_owner
def _burn(self, account:Address, amount:int) -> None:
```

####\_approve
Sets the allowance value given by the `owner` to the `spender`.
```Python
def _approve(self, owner:Address, spender:Address, value:int) -> None:
```

####\_beforeTokenTransfer
Called before transfer of tokens to check if it is valid. If `_from` and `_to` are both non zero, `_value` number of tokens of `_from` will be transferred to `_to`. If `_from` is zero `_value` tokens will be minted to `_to`. 		If `_to` is zero `_value` tokens will be destroyed from `_from`. Both `_from` and `_to` are never both zero at once.
```Python
def _beforeTokenTransfer(self, _from: Address, _to: Address,_value: int) -> None:
```

## Exceptions

####InsufficientBalanceError
Raised when the sender has less balance than the value to be transferred during transfer operation.

####ZeroValueError
Raised when 0 tokens is to be transferred, minted or burned.

####InvalidNameError
Raised when the length of the `_tokenName` and `_symbolName` of the token is 0.

####OverCapLimit
Raised when total supply exceeds the cap limit.

####InsufficientAllowanceError
Raised when amount to be transferred exceeds the allowance amount.


## Implementation
* [IRC2 Token](https://github.com/OpenDevICON/odi-contracts/blob/test-fixed/ODIContracts/tokens/IRC2.py "IRC2")