# IRC2 Capped
IRC2 Capped tokens are IRC2-compatible tokens with one added feature: there is a limit to the number of tokens that can be created. Standard IRC2 tokens don't have this feature, which makes them a fixed supply tokens.

Cap limit can be set only by owner of the contract which is set when the SCORE is first deployed or updated.

## Methods

####cap
Returns the cap limit of the token.
```Python
@external(readonly=True)
def cap(self) -> str:
```

## Internal Function 

####\_beforeTokenTransfer
Checks if the total supply exceeds `cap` limit
```Python
def _beforeTokenTransfer(self, _from:Address, _to:Address, _value:int) -> None:
```

## Implementation
* [IRC2Capped Token](https://github.com/OpenDevICON/odi-contracts/blob/test-fixed/ODIContracts/tokens/IRC2capped.py "IRC2Capped")
