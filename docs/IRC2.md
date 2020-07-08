# IRC2 Token

A token is a representation of something in blockchain. This something can be money, services, shares in a company or anything. By representing something as tokens, we can allow smart contract to interact with them. 

IRC2 token is a standard token equivalent to [ERC20]() for [ICON](https://icon.foundation/?lang=en) blockchain. It helps to keep track of [fungible](https://en.wikipedia.org/wiki/Fungibility) tokens. 

There are a few core contracts to implement IRC2 token.
* IIRC2: Interface IRC2 methods should confirm into.
* IRC2: The base implementation of IRC2 contract.

There are multiple other extensions.
* IRC2Mintable: To create token supply.
* IRC2Capped: Total supply cannot exceed the cap amount. 
* IRC2Burnable: To destroy the tokens.
* IRC2Pausable: To pause token operation for all users.

##CORE<hr>
###IRC2 
####Methods
**name**
Returns the name of the token. eg. `SampleToken`
```Python
	@external(readonly=True)
	def name(self) -> str:
```
**symbol**
Returns the symbol of the token. eg. `ST`
```Python
	@external(readonly=True)
	def symbol(self) -> str:
```

**decimals**
Returns the number of decimals in the token. It is 18 by default. eg. `18`
```Python
	@external(readonly=True)
	def decimals(self) -> str:
```

**totalSupply**
Returns the total number of tokens in existence.
```Python
	@external(readonly=True)
	def totalSupply(self) -> int:
```
**balanceOf**
Returns the amount of tokens owned by the account with address `account`.
```Python
	@external
	def balanceOf(self,account: Address) -> int:
```

**transfer**
Transfers `_value` amount of tokens to the address `_to` from `self.msg.sender` and fires Transfer eventlog. `self.msg.sender` should have enough balance to transfer to `_to`. If `_to` is a contract, this function must invoke the function `tokenFallback(Address, int, bytes)` in `_to`. If the `tokenFallback` function is not implemented in `_to` (receiver contract), then the transaction must fail and the transfer of tokens should not occur. If `_to` is an externally owned address, then the transaction must be sent without trying to execute `tokenFallback` in `_to`.  `_data` can be attached to this token transaction. `_data` can be empty.
```Python
	@external
	def transfer(self, _to: Address, _value: int, _data: bytes = None) -> bool:
```






