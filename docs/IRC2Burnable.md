# IRC2 Burnable
IRC2 Burnable tokens are IRC2-compatible tokens with one added feature: tokens can be burned(destroyed) at any time and reduced from the total supply. Standard IRC2 tokens don't have this feature, which makes them a fixed supply tokens.

Tokens can be burnt only by owner of the contract which is set when the SCORE is first deployed or updated.

## Methods

####burn
Destroys `_amount` number of tokens from the caller account. Decreases the balance of that account and total supply. Current implementation allows only the owner of SCORE to call this method.
```Python
@external
def burn(self, _amount: int) -> None:
```
<hr>

####burnFrom
Destroys `_amount` number of tokens from the `_account`. Decreases the balance of `_account` and total supply. Current implementation allows only the owner of SCORE to call this method.
```Python
@external
def burnFrom(self, _account: Address, _amount: int) -> None:
```

## Implementation
* [IRC2Burnable Token](https://github.com/OpenDevICON/odi-contracts/blob/test-fixed/ODIContracts/tokens/IRC2burnable.py "IRC2Burnable")