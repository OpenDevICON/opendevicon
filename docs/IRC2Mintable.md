# IRC2 Mintable
IRC2 Mintable tokens are IRC2-compatible tokens with one added feature: new tokens can be created at any time and added to total supply. Standard IRC2 tokens don't have this feature, which makes them a fixed supply tokens.

New tokens can be minted only by owner of the contract which is set when the SCORE is first deployed or updated.

____


## Methods

#### mint: 
Creates `_amount` number of tokens, and assigns to caller account. Increases the balance of that account and total supply. Only owner of SCORE can call this method.
```
@external
def mint(self, _amount: int) -> None:
```
<hr>

#### mintTo: 
Creates `_amount` number of tokens, and assigns to `_account`. Increases the balance of `_account` and total supply. Only owner of SCORE can call this method.

```
@external
def mintTo(self, _account: Address, _amount: int) -> None:
```

---

## Implementation
* [IRC2Mintable Token](https://github.com/OpenDevICON/odi-contracts/blob/test-fixed/ODIContracts/tokens/IRC2mintable.py "IRC2Mintable")