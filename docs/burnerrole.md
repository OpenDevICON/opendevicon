# Burner Role


##Methods

####isBurner
Returns if `_account` is a burner, i.e. has permission to burn tokens.
```python
def isBurner(self, _account: Address) -> bool:
```

####burnersList
Returns the list of burners. It calls the internal function `_burnersList` from roles.
```python
def burnersList(self):
```

####addBurner
Adds `_account` to the burners list. Then, the `_account` has permission to burn the tokens. Can be called only by the owner of the SCORE.
```py
def addBurner(self, _account: Address) -> bool:
```

####removeBurner
Removes `_account` from the burners list. Then, the `_account` no longer has permission to burn the tokens. Can be called only by the owner of the SCORE.
```py
def removeBurner(self, _account: Address) -> bool:
```

####renounceBurner
Can be called only by burners. Renounces the position of address who executes this method as a burner. Then, the address no longer has permission to burn the tokens. 
```py
def renounceBurner(self) -> bool:
```


##EventLogs

####BurnerAdded
Triggered when a new burner is added.
```python
@eventlog(indexed=0)
  	def BurnerAdded(self, _account: Address):
```

####BurnerRemoved
Triggered when a burner is removed.
```python
@eventlog(indexed=0)
  	def BurnerRemoved(self, _account: Address):
```

##Internal Functions

#### \_addBurner
Adds `_account` to the burners list and emits BurnerAdded event.
```py
def _addBurner(self, _account: Address) -> None:
```
#### \_removeBurner
Removes `_account` from the burners list and emits BurnerRemoved event.
```py
def _removeBurner(self, _account: Address) -> None:
```


## Implementation
* [Burner Role](https://github.com/OpenDevICON/odi-contracts/blob/development/ODIContracts/access/role/BurnerRole.py "BurnerRole")