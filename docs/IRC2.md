#IRC2 
##Methods
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

####allowance
Returns the number of tokens that the `spender` will be allowed to spend on behalf of `owner`.
```Python
	@external
	def _allowance(self, owner: Address, spender: Address) -> int:
```

####approve
Sets the allowance value given by the `owner` to the `spender`. Returns a boolen value to check if the operation was successful.
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










### Eventlogs

#### Transfer
Must trigger on any successful token transfers.
```python
@eventlog(indexed=3)
def Transfer(self, _from: Address, _to: Address, _value: int, _data: bytes):
    pass
```

### Token Fallback

A function for handling token transfers, which is called from the token contract, when a token holder sends tokens. `_from` is the address of the sender of the token, `_value` is the amount of incoming tokens, and `_data` is arbitrary attached data. It works by analogy with the fallback function of the normal transactions and returns nothing.
```python
@external
def tokenFallback(self, _from: Address, _value: int, _data: bytes):
```




