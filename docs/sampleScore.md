# Sample SCORE

Before showing the usage of PythonSDK in Jupyter Notebook, we will require a SCORE to be interacted with. For this documentation we choose to use one of the SCORE having highest number of transactions in ICON blockchain ie Dice SCORE of [ICONbet](https://iconbet.io/).


## Dice SCORE
```py
# %load dice/dice.py
from iconservice import *

TAG = "DICE"
UPPER_LIMIT = 99
LOWER_LIMIT = 0
MAIN_BET_MULTIPLIER = 98.5
SIDE_BET_MULTIPLIER = 95
BET_MIN = 1000000000000000
SIDE_BET_TYPES = ["digits_match", "icon_logo1", "icon_logo2"]
SIDE_BET_MULTIPLIERS = {"digits_match": 9.5, "icon_logo1": 5, "icon_logo2": 95}
BET_LIMIT_RATIOS_SIDE_BET = {"digits_match": 1140, "icon_logo1": 540, "icon_logo2": 12548}
MINIMUM_TREASURY = 250000





class Dice(IconScoreBase):
    _GAME_ON = "game_on"

    def __init__(self, db: IconScoreDatabase) -> None:
        super().__init__(db)
        self._game_on = VarDB(self._GAME_ON, db, value_type=bool)

    @eventlog(indexed=2)
    def BetSource(self, _from: Address, timestamp: int):
        pass

    @eventlog(indexed=3)
    def PayoutAmount(self, payout: int, main_bet_payout: int, side_bet_payout: int):
        pass

    @eventlog(indexed=3)
    def BetResult(self, spin: str, winningNumber: int, payout: int):
        pass

    @eventlog(indexed=2)
    def FundTransfer(self, recipient: Address, amount: int, note: str):
        pass

    def on_install(self) -> None:
        super().on_install()
        self._game_on.set(False)

    def on_update(self) -> None:
        super().on_update()


    @external
    def toggle_game_status(self) -> None:
    
        if self.msg.sender != self.owner:
            revert('Only the owner can call the toggle_game_status method')
        self._game_on.set(not self._game_on.get() )


    @external(readonly=True)
    def get_game_status(self) -> bool:
        return self._game_on.get()

    @external(readonly=True)
    def get_side_bet_multipliers(self) -> dict:
        return SIDE_BET_MULTIPLIERS


    def get_random(self, user_seed: str = '') -> float:
        seed = (str(bytes.hex(self.tx.hash)) + str(self.now()) + user_seed)
        spin = (int.from_bytes(sha3_256(seed.encode()), "big") % 100000) / 100000.0
        return spin

    @payable
    @external
    def call_bet(self, upper: int, lower: int, user_seed: str = '', side_bet_amount: int = 0,
                 side_bet_type: str = '') -> None:
        """
        Main bet function. It takes the upper and lower number for bet. Upper and lower number must be in the range
        [0,99]. The difference between upper and lower can be in the range of [0,95].

        :param upper: Upper number user can bet. It must be in the range [0,99]
        :type upper: int
        :param lower: Lower number user can bet. It must be in the range [0,99]
        :type lower: int
        :param user_seed: 'Lucky phrase' provided by user, defaults to ""
        :type user_seed: str,optional
        :param side_bet_amount: Amount to be used for side bet from value sent to this function, defaults to 0
        :type side_bet_amount: int,optional
        :param side_bet_type: side bet types can be one of this ["digits_match", "icon_logo1","icon_logo2"], defaults to
         ""
        :type side_bet_type: str,optional
        """
        return self.__bet(upper, lower, user_seed, side_bet_amount, side_bet_type)

    def __bet(self, upper: int, lower: int, user_seed: str, side_bet_amount: int, side_bet_type: str) -> None:
        side_bet_win = False
        side_bet_set = False
        side_bet_payout = 0
        self.BetSource(self.tx.origin, self.tx.timestamp)

        #condition checks
        if not self._game_on.get():
            revert(f'Game not active yet.')
        
        if not (0 <= upper <= 99 and 0 <= lower <= 99):
            revert(f'Invalid bet. Choose a number between 0 to 99')
        if not (0 <= upper - lower <= 95):
            revert(f'Invalid gap. Choose upper and lower values such that gap is between 0 to 95')
        if (side_bet_type == '' and side_bet_amount != 0) or (side_bet_type != '' and side_bet_amount == 0):
            revert(f'should set both side bet type as well as side bet amount')
        if side_bet_amount < 0:
            revert(f'Bet amount cannot be negative')
        if side_bet_type != '' and side_bet_amount != 0:
            side_bet_set = True
            if side_bet_type not in SIDE_BET_TYPES:
                revert(f'Invalid side bet type.')
            side_bet_limit = MINIMUM_TREASURY // BET_LIMIT_RATIOS_SIDE_BET[side_bet_type]
            if side_bet_amount < BET_MIN or side_bet_amount > side_bet_limit:
                revert(f'Betting amount {side_bet_amount} out of range ({BET_MIN} ,{side_bet_limit}).')
            side_bet_payout = int(SIDE_BET_MULTIPLIERS[side_bet_type] * 100) * side_bet_amount // 100


        main_bet_amount = self.msg.value - side_bet_amount
        gap = (upper - lower) + 1
        if main_bet_amount == 0:
            Logger.debug(f'No main bet amount provided', TAG)
            revert(f'No main bet amount provided')

        main_bet_limit = (MINIMUM_TREASURY * 1.5 * gap) // (68134 - 681.34 * gap)
        if main_bet_amount < BET_MIN or main_bet_amount > main_bet_limit:
            revert(f'Main Bet amount {main_bet_amount} out of range {BET_MIN},{main_bet_limit} ')
        main_bet_payout = int(MAIN_BET_MULTIPLIER * 100) * main_bet_amount // (100 * gap)
        payout = side_bet_payout + main_bet_payout
        if self.icx.get_balance(self.address) < payout:
            revert('Not enough in treasury to make the play.')
        spin = self.get_random(user_seed)
        winningNumber = int(spin * 100)
        if lower <= winningNumber <= upper:
            main_bet_win = True
        else:
            main_bet_win = False
        if side_bet_set:
            side_bet_win = self.check_side_bet_win(side_bet_type, winningNumber)
            if not side_bet_win:
                side_bet_payout = 0
        main_bet_payout = main_bet_payout * main_bet_win
        payout = main_bet_payout + side_bet_payout
        self.BetResult(str(spin), winningNumber, payout)
        self.PayoutAmount(payout, main_bet_payout, side_bet_payout)
        if main_bet_win or side_bet_win:
            try:
                self.icx.transfer(self.msg.sender,payout)
            except BaseException as e:
                revert('Network problem. Winnings not sent. Returning funds.')

    # check for bet limits and side limits
    def check_side_bet_win(self, side_bet_type: str, winning_number: int) -> bool:
        """
        Checks the conditions for side bets are matched or not.
        :param side_bet_type: side bet types can be one of this ["digits_match", "icon_logo1","icon_logo2"], defaults to
         ""
        :type side_bet_type: str,optional
        :param winning_number: winning number returned by random function
        :type winning_number: int
        :return: Returns true or false based on the side bet type and the winning number
        :rtype: bool
        """
        if side_bet_type == SIDE_BET_TYPES[0]:  # digits_match
            return winning_number % 11 == 0
        if side_bet_type == SIDE_BET_TYPES[1]:  # for icon logo1 ie for numbers having 1 zero in it
            return str(0) in str(winning_number) or winning_number in range(1, 10)
        if side_bet_type == SIDE_BET_TYPES[2]:  # for icon logo2 ie for 0
            return winning_number == 0

    @payable
    def fallback(self):
        if self.msg.sender != self.owner:
            revert("Treasury can only be filled by the SCORE owner")
```