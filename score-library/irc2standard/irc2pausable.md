# IRC2 Pausable

IRC2 Pausable tokens are IRC2-compatible tokens with one added feature: token transfers can be paused and unpaused as per the requirement. Standard IRC2 tokens don't have this feature.

Only owner of SCORE and pausers can pause and unpause token transfers. When paused, no token transfers can take place - this also includes minting, burning.

{% hint style="warning" %}
Note: When the tokens are paused, token transfers are NOT suspended for later. Token transfers simply cannot proceed.
{% endhint %}

## Methods

### paused

Returns True if the token transfer has been paused.

```text
@external(readonly=True)
def paused(self) -> bool:
```

### pause

Pauses the token transfers. This method can be executed only by the owner of SCORE and pausers when the SCORE is not in paused state. It emits Paused event.

```text
@external
@whenNotPaused
@only_owner
def pause(self):
```

### unpause

Unpauses the token transfers. This method can be executed only by the owner of SCORE when the SCORE and pausers is already in paused state. It emits Unpaused event.

```text
@external
@whenPaused
@only_owner
def unpause(self):
```

## Events

### Paused

```text
@eventlog(indexed=1)
def Paused(self, status:bool):
```

## Internal Functions

### \_beforeTokenTransfer

Checks if the SCORE is is paused state before transferring tokens.

```python
def _beforeTokenTransfer(self, _from:Address, _to:Address, _value:int) -> None:
```

## Exceptions

### AlreadyPausedException:

This exception is raised when the SCORE is already in paused state and token transfers calls are called. It is also raised when the pause method is called even though the SCORE is already in paused state.

### AlreadyUnpausedException:

This exception is raised when unpause method is called when the SCORE is already in unpaused state.

## Implementation

* [IRC2Pausable Token](https://github.com/OpenDevICON/odi-contracts/blob/test-fixed/ODIContracts/tokens/IRC2pausable.py,)

