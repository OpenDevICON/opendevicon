# Score Methods

Let us go through some of the SCORE methods used in Dice game so that we can figure out what actually happens when we make a call to them.

## Eventlog Methods
Eventlogs are dummy methods without any body. They are used to provide information about the output of the transaction through their arguments. They are called within SCORE codes during the execution of a transaction to include custom event logs in its **TxResult** as event logs.

**FundTransfer Eventlog**
This method is used to show any transfer of ICX inside the SCORE.

```py
@eventlog(indexed=2)
    def FundTransfer(self, recipient: Address, amount: int, note: str):
        pass
```
**PayoutAmount Eventlog**
This method is used to provide the payout information in case the user wins.

```py
@eventlog(indexed=3)
    def PayoutAmount(self, payout: int, main_bet_payout: int, side_bet_payout: int):
        pass
```
**BetResult Eventlog**
This method is used to show the winning number.
```py
@eventlog(indexed=1)
    def BetResult(self, spin: str, winningNumber: int):
        pass
```
## External Methods
Methods decorated with **@external** can be called from outside the contract. Basically a call to these methods
**toggle_game_status Method**
This method is used to turn the game on and off.
{% hint style="info" %}
Only the SCORE owner can make this method call
{% endhint %}
```py
@external
def toggle_game_status(self) -> None:

    if self.msg.sender != self.owner:
        revert('Only the owner can call the toggle_game_status method')
    self._game_on.set(not self._game_on.get() )
```

## Payable Methods
Methods decorated with **@payable** are permitted to receive incoming ICX.
**call_bet Method**
This method is used to place your bet for playing the game.

Method defination:-
```py
@payable
@external
def call_bet(self, upper: int, lower: int, user_seed: str = '', side_bet_amount: int = 0,
                side_bet_type: str = '') -> None:
```

## Readonly Methods
Methods decorated with **@external(readonly=True)** will have read-only access to the state database.They can be used to provide the required contract information.
{% hint style="info" %}
No transaction is performed while calling readonly methods
{% endhint}
**get_game_status Method**
This method is used to get the status of the game.
```py
@external(readonly=True)
    def get_game_status(self) -> bool:
        return self._game_on.get()
```

Methods that we will be using our Juypter Notebook file are:-
-toggle_game_status
-get_game_status
-call_bet
