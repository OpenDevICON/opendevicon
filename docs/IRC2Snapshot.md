# IRC2Snapshot
IRC2Snapshot tokens are IRC2-compatible tokens with one added feature: snapshots mechanism. When a snapshot is created, the balances and total supply at the time are recorded for later access.

This can be used to safely create mechanisms based on token balances such as trustless dividends or weighted voting. In naive implementations it’s possible to perform a "double spend" attack by reusing the same balance from different accounts. By using snapshots to calculate dividends or voting power, those attacks no longer apply. 

Snapshots are created by the internal \_snapshot function, which will emit the Snapshot event and return a snapshot id. To get the total supply at the time of a snapshot, call the function totalSupplyAt with the snapshot id. To get the balance of an account at the time of a snapshot, call the balanceOfAt function with the snapshot id and the account address.

## Methods

#### snapshot
Creates a new snapshot and sets its snapshot id. Also emits a Snapshot event that contains the same id.
```
@external
def snapshot(self) -> None:
```

#### balanceOfAt
Returns the balance of `_account` at the time when `_snapshot_id` was created.
```
@external(readonly=True)
def balanceOfAt(self, _account: Address, _snapshot_id: int) -> int:
```

#### totalSupplyat
Returns the totalSupply of at the time when `_snapshot_id` was created.
```
@external(readonly=True)
def totalSupplyat(self, _snapshot_id: int) -> int:
```

#### transfer
Extends the default IRC2 token transfer behaviour with `_updateAccountSnapshot` internal method.
```
@external
def transfer(self,_to: Address, _value: int, _data: bytes = None) -> None:
```

#### mint
Extends the default IRC2Mintable token minting behaviour with `_updateAccountSnapshot` and `_updateTotalSupplySnapshot` internal methods.
```
@external
def mint(self, _amount: int) -> None:
```

#### burn
Extends the default IRC2Burnable token burning behaviour with `_updateAccountSnapshot` and `_updateTotalSupplySnapshot` internal methods.
```
@external
def burn(self, _amount: int) -> None:
```

---

## Events

#### Snapshot
Emits a snapshot event.
```Python
	@eventlog(indexed=1)
	def Snapshot(self, _id: int) -> None:
```

---


## Internal Functions
#### \_snapshot
Creates a new snapshot and sets its snapshot id
```Python
def _snapshot(self) -> None:
```

#### \_transfer
Changes the account snapshot after transfer operation.
```Python
def _transfer(self, _from: Address, _to: Address, _value: int, _data) -> None:
```

#### \_mint
Changes the total supply snapshot and account snapshot after mint operation.
```Python
def _mint(self, _account: Address, _value: int) -> None:
```

#### \_burn
Changes the total supply snapshot and account snapshot after burn operation.
```Python
_burn(self, _account: Address, _value: int) -> None:
```

#### \_updateAccountSnapshot
Updates snapshot of a account after `transfer`, `mint` or `burn` operation.
```Python
_updateAccountSnapshot(self, _account: Address) -> None:
```

#### \_updateTotalSupplySnapshot
Updates total supply snapshot after `mint` or `burn` operation.
```Python
_updateTotalSupplySnapshot(self) -> None:
```

---

## Implementation

* [IRC2Snapshot Token](https://github.com/OpenDevICON/odi-contracts/blob/test-fixed/ODIContracts/tokens/IRC2snapshot.py, "IRC2Snapshot")