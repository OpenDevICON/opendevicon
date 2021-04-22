# Pauser Role

## Methods

### isPauser

Returns if `_account` is a pauser, i.e. has permission to pause and unpause tokens.

```python
def isPauser(self, _account: Address) -> bool:
```

### pausersList

Returns the list of pausers. It calls the internal function `_pausersList` from roles.

```python
def pausersList(self):
```

### addPauser

Adds `_account` to the pausers list. Then, the `_account` has permission to pause and unpause the tokens. Can be called only by the owner of the SCORE.

```python
def addPauser(self, _account: Address) -> bool:
```

### removePauser

Removes `_account` from the pausers list. Then, the `_account` no longer has permission to pause and unpause the tokens. Can be called only by the owner of the SCORE.

```python
def removePauser(self, _account: Address) -> bool:
```

### renouncePauser

Can be called only by the pausers. Renounces the position of address who executes this method as a pauser. Then, the `_account` no longer has permission to pause and unpause the tokens..

```python
def renouncePauser(self) -> bool:
```

## EventLogs

### PauserAdded

Triggered when a new pauser is added.

```python
@eventlog(indexed=0)
      def PauserAdded(self, _account: Address):
```

### PauserRemoved

Triggered when a pauser is removed.

```python
@eventlog(indexed=0)
      def PauserRemoved(self, _account: Address):
```

## Internal Functions

### \_addPauser

Adds `_account` to the pausers list and emits PauserAdded event.

```python
def _addPauser(self, _account: Address) -> None:
```

### \_removePauser

Removes `_account` from the pausers list and emits PauserRemoved event.

```python
def _removePauser(self, _account: Address) -> None:
```

## Implementation

* [Pauser Role](https://github.com/OpenDevICON/odi-contracts/blob/development/ODIContracts/access/role/PauserRole.py)

