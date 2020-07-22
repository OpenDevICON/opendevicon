# Minter Role


##Methods

####isMinter
Returns if `_account` is a minter, i.e. has permission to mint tokens.
```python
def isMinter(self, _account: Address) -> bool:
```

####mintersList
Returns the list of minters. It calls the internal function `_mintersList` from roles.
```python
def mintersList(self) -> None:
```

####addMinter
Adds `_account` to the minters list. Then, the `_account` has permission to mint the tokens. Can be called only by the owner of the SCORE.
```py
def addMinter(self, _account: Address) -> bool:
```

####removeMinter
Removes `_account` from the minters list. Then, the `_account` no longer has permission to mint the tokens. Can be called only by the owner of the SCORE.
```py
def removeMinter(self, _account: Address) -> bool:
```

####renounceMinter
`_account` renounces the position as a minter. Then, the `_account` no longer has permission to mint the tokens. Can be called only by the minters.
```py
def renounceMinter(self) -> bool:
```


##EventLogs

####MinterAdded
Triggered when a new minter is added.
```python
@eventlog(indexed=0)
  	def MinterAdded(self, _account: Address):
```

####MinterRemoved
Triggered when a minter is removed.
```python
@eventlog(indexed=0)
  	def MinterRemoved(self, _account: Address):
```

##Internal Functions

#### \_addMinter
Adds `_account` to the minters list and emits MinterAdded event.
```py
def _addMinter(self, _account: Address) -> None:
```
#### \_removeMinter
Removes `_account` from the minters list and emits MinterRemoved event.
```py
def _removeMinter(self, _account: Address) -> None:
```


## Implementation
* [Minter Role](https://github.com/OpenDevICON/odi-contracts/blob/development/ODIContracts/access/role/MinterRole.py "MinterRole")