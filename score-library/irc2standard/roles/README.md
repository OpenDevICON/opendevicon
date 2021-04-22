# Access

## Roles

Library for managing the address assigned to a role. During initialization, the score deployer is added to the minterslist, pauserslist and burnerlist, i.e. the deployer can perform all these operations from the start.

## Internal functions

### add

It is used to assign a address to a specific role. It is an internal function, which is called from the function in `minterrole`, `burnerrole` and `pauserrole`. The role is checked, and that role is assigned to the address.

```python
def add(self, Role: str, _account: Address):
```

### remove

It is used to remove a address from a specific role. It is an internal function, which is called from the function in `minterrole`, `burnerrole` and `pauserrole`. The role is checked, then the address is checked to see if it has that role. Then, the address is removed from that role.

```python
def remove(self, Role: str, _account: Address):
```

### has

It is used to check if a address is assigned to a specific role. It is an internal function, which is called from the function in `minterrole`, `burnerrole` and `pauserrole`.

```python
def has(self, Role: str, _account: Address) -> bool:
```

### \_mintersList

It is used to return the list of the minters, i.e. the addresses that were given permission to mint the tokens.

```python
def _mintersList(self):
```

### \_burnersList

It is used to return the list of the burners, i.e. the addresses that were given permission to burn the tokens.

```python
def burnersList(self):
```

### \_pausersList

It is used to return the list of the pausers, i.e. the addresses that were given permission to pause and unpause the operations.

```python
def pausersList(self):
```

## Implementation

* [Roles](https://github.com/OpenDevICON/odi-contracts/blob/development/ODIContracts/access/roles.py)

