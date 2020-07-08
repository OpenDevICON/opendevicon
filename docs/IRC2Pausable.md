# IRC2Pausable
IRC2 Pausable tokens are IRC2-compatible tokens with one added feature: token transfers can be paused and unpaused as per the requirement. Standard IRC2 tokens don't have this feature.

Only owner of SCORE can pause and unpause token transfers. When paused, no token transfers can take place - this also includes minting, burning. 

> Note: When the tokens are paused, token transfers are NOT suspended for later. Token transfers simply cannot proceed.

---

## Methods

#### paused
Returns if the token transfer has been paused.
```
@external(readonly=True)
def paused(self) -> bool:
```

#### pause
Pauses the token transfers. This method can be executed only by the owner of SCORE when the SCORE is not already in paused state. It emits Paused event.
```
@external
@whenNotPaused
@only_owner
def pause(self):
```

#### unpause
Unpauses the token transfers. This method can be executed only by the owner of SCORE when the SCORE is already in paused state. It emits Unpaused event.
```
@external
@whenPaused
@only_owner
def unpause(self):
```
---

## Events

#### Paused
```
@eventlog(indexed=1)
def Paused(self, by:Address):
```

#### Unpaused
```
@eventlog(indexed=1)
def Unpaused(self, by:Address):

```
---

## Internal Functions
#### _beforeTokenTransfer
Checks if the SCORE is is paused state before transferring tokens.
```Python
def _beforeTokenTransfer(self, _from:Address, _to:Address, _value:int) -> None:
```

---



## Exceptions

#### AlreadyPausedException:
This exception is raised when the SCORE is already is paused state and token transfers calls are called. It is also raised when the pause method is called even though the SCORE is already in paused state. 

#### AlreadyUnpausedException:
This exception is raised when unpause method is called when the SCORE is already in unpaused state.