#Idol Token
Idol token is a sample NFT built on ICON network with the IRC3 standard.There are some custom methods in the score to make things easier and avoid redundancy.The specifications of the token and the implementation are discussed in detail below.

##Variables and types
The various variables used and their types along with the initialization can be found in the code below:
```py
_OWNER_ADDRESS="owner_address"
_TOKEN_APPROVED="token_approved"
_IDOL_OWNER="idol_owner"
_OWNER_IDOL_COUNT="owner_idol_count"
_IDOL_LIST="idol_list"
_IDOL="idol"
_ZERO_ADDRESS = Address.from_prefix_and_int(AddressPrefix.EOA, 0)
 
def __init__(self, db: IconScoreDatabase) -> None:
	super().__init__(db)
	self._owner_address = VarDB(self._OWNER_ADDRESS, db, value_type=Address)
	self._token_approved = DictDB(self._TOKEN_APPROVED, db, value_type=Address)
	self._idolOwner = DictDB(self._IDOL_OWNER, db, value_type=Address)
	self._ownerToIdolCount = DictDB(self._OWNER_IDOL_COUNT, db, value_type=int)
	self._idolRegister = ArrayDB(self._IDOL_LIST, db, value_type=str)
	self._idols = DictDB(self._IDOL, db, value_type=str, depth=2)
```

## Methods

####name
Returns the name of the token. Any name can be given to a NFT, in our case we have given “IdolToken” as name.
```py
@external(readonly=True)
def name(self) -> str:
	return "IdolToken"
```

####symbol
Returns the symbol of a token. Any abbreviated form can be given as symbol, token symbol with all letters in uppercase is a general trend while giving the symbol.
```py
@external(readonly=True)
def symbol(self) -> str:
	return "IDOL"
```

####balanceOf
Returns the number of `Idol Tokens` owned by any `_owner`.The function takes `_owner` of type address as parameter and returns the total number of `Idol Tokens` belonging to that address. NFTs assigned to the zero address are considered invalid, so this function should raise error for queries about the zero address.
```py
@external(readonly=True)
def balanceOf(self, _owner: Address) -> int:
	if _owner is None or  self._is_zero_address(_owner):
    	revert("Invalid owner address")
	return self._ownerToIdolCount[_owner]
```

####ownerOf
Returns the owner of the `Idol Token` of a given id. Throws error if the given `_tokenId` is invalid.
```py
@external(readonly=True)
def ownerOf(self, _tokenId: int) -> Address:
	if not (self._id_validity(_tokenId)):
		revert("Invalid tokenId")
	return self._idolOwner[str(_tokenId)]
```

####getApproved
Returns the approved address for a single `_tokenId`. If there is none, return the zero address. It should throw an error if `_tokenId` is not a valid id. The token needs to be approved by the owner before it is transferred to other user and this method helps in keeping the track whether a token was approved by the current holder or not.
```py
@external(readonly=True)
def getApproved(self, _tokenId: int) -> Address:
    if not (self._id_validity(_tokenId)):
    	revert("Invalid tokenId")
    addr = self._token_approved[str(_tokenId)]
    if addr is None:
    	return self._ZERO_ADDRESS
	return addr
```

####approve
Allows `_to` to change the ownership of `_tokenId` from your account. The zero address indicates there is no approved address. Throws unless `self.msg.sender` is the current `_tokenId` owner. Only the current owner of the `Idol Token` can approve for the transfer. The current owner can approve their car token to any other address except the zero address.
```py
@external
def approve(self, _to: Address, _tokenId: int):
	tokenOwner = self._idolOwner[str(_tokenId)]
	if tokenOwner != self.msg.sender:
		raise IconScoreException("approve : sender does not own the token")
	if _to==tokenOwner:
		raise IconScoreException("approve : cant approve to token owner")
	self._token_approved[str(_tokenId)] = _to
	self.Approval(self.msg.sender, _to, _tokenId)
```

####transfer
Transfers the ownership of a idol token from the current owner of the token to other user. This method must fire an event named Transfer(this will discussed later in another section). This method should throw error in case `self.msg.sender`  is not the current owner of the token, if the receiving address of the token is a zero address and if the `_tokenId`  is not a valid token id. Then, it calls the \_transfer internal function.
```py
@external
def transfer(self, _to: Address, _tokenId: int):
	idolOwner = self.ownerOf(_tokenId)
	if idolOwner != self.msg.sender:
		raise IconScoreException("transfer : sender does not own the token")
	if self._is_zero_address(_to):
		raise IconScoreException("transfer : receiver cant be a zero address")
	approved = self.getApproved(_tokenId)
	if approved != _to:
		raise IconScoreException("transfer : _to is not approved")
	self._transfer(self.msg.sender,_to,_tokenId)
```

####transferFrom
Transfers the ownership of a idol token from the current owner of the token to other user. This method must fire an event named Transfer. It throws error unless `self.msg.sender` is the current owner or the approved address for the car token, throws error if `_from` is not the current owner, throws error if `_to` is the zero address and throws error  if `_tokenId` is not a valid idol token(\_tokenId).

```py
@external
def transferFrom(self, _from: Address, _to: Address, _tokenId: str):
	idolOwner = self.ownerOf(_tokenId)
	if idolOwner != _from:
		raise IconScoreException("transfer : _from does not own the token")
	if idolOwner!=self.msg.sender and self._token_approved[str(_tokenId)]!=self.msg.sender:
		raise IconScoreException("transfer : unauthorised user for trasfer")
	approved = self.getApproved(_tokenId)
	if approved != _to:
		raise IconScoreException("transfer : _to is not approved")
	self._transfer(_from,_to,_tokenId)
```

####create_idol
Used to create a new `Idol Token`. The token will have `_name`, `_age`, `_gender` and `_ipfs_handle` as parameters. `_ipfs_handle` is the ipfs hash of the image of the idol.
```py
@external
def create_idol(self, _name: str, _age: str, _gender: str, _ipfs_handle: str):
	# Use token index as unique identifier to add attributes to DictDB
	attribs = {"name": _name, "age": _age, "gender": _gender, "ipfs_handle": _ipfs_handle}
	_tokenId = str(self.totalSupply() + 1)
	self._idolRegister.put(_tokenId)
	self._idolOwner[_tokenId] = self.msg.sender
	self._ownerToIdolCount[self.msg.sender] += 1
	self._idols[_tokenId]["attributes"]=json_dumps(attribs)
```

####get_idol
Returns the `Idol Token` which has the id `_tokenId`. 
```py
@external(readonly=True)
def get_idol(self, _tokenId: int) -> dict:
	if not self._id_validity(_tokenId):
		IconScoreException("idol details : invalid id")
	idol_attribs = {}
	idol_attribs=json_loads(self._idols[str(_tokenId)]["attributes"])
	idol_attribs["owner"]=self.ownerOf(_tokenId)
	return idol_attribs
```
####get_tokens_of_owner
Returns all the `Idol Tokens` owned by the `_owner`.
```py
@external(readonly=True)
def get_tokens_of_owner(self, _owner: Address) -> dict:
	idol_token_list = []
	for _id in self._idolRegister:
		if self._idolOwner[str(_id)] == _owner:
			idol_token_list.append(str(_id))

	return {'idols': idol_token_list}
```

## Internal Function 

####\_is_zero_address
It is a custom method to check whether a given `_address` is a zero address or not, it returns True if the `_address` is a zero address and vice-versa. Its code implementation is below:
```py
def _is_zero_address(self, _address: Address) -> bool:
    # Check if address is zero address
    if _address == self._ZERO_ADDRESS:
        return True
    return False
```

####\_id_validity
It is a custom method which check whether a token with the given id `_tokenId` exists or not. Its implementation is below:
```py
def _id_validity(self,_tokenId)-> bool:
    if str(_tokenId) not in self._idolRegister:
    	return False
    return True
```

####\_transfer
It is a custom method which transfer the ownership from owner address to the receiving address if the transfer details are valid. The same transfer logic is used in `transfer` and `transferFrom` also so, it would be better to make a custom method to avoid redundancy. The implementation of this method is below:
```py
def _transfer(self,_from:Address,_to:Address,_tokenId:int):
	self._idolOwner[str(_tokenId)]=_to
	self._ownerToIdolCount[_to]+=1
	self._ownerToIdolCount[_from]-=1
	del self._token_approved[str(_tokenId)]
	self.Transfer(_from,_to,_tokenId)
```

## Eventlogs

####Transfer
This event is triggered when a successful token transfer happens.
```py
@eventlog(indexed=3)
def Transfer(self, _from: Address, _to: Address, _tokenId: int):
	pass
```

####Approve:
This event is triggered when token is successfully approved for transfer from the current owner, i.e. `approve` method is successfully executed 
```py
@eventlog(indexed=3)
def Approval(self, _owner: Address, _approved: Address, _tokenId: int):
	pass
```


## Differences between IRC2 tokens and IRC3 tokens:
|IRC2|IRC3|
|:---|:---|
|It has standard interface for a fungible token|It has standard interface for a non-fungible token|
|Owner can directly send their token to any address without calling the approve method.|The owner needs to approve the tokenId of a token before transferring the token.|
|The tokens can have decimal values.(eg a person can have 2.34444 tokens)|The tokens cannot have decimal values.They must be in integer.(eg:a person cannot have 2.34444 tokens ,he can have 2 or 3 or any other integer value)|
|Each  token don’t have specific token_id|Each token have a unique token_id|
|Token fallback mechanism is adopted|approve/transferFrom method is adopted|
|Inspired by ERC223 token standard|Inspired by ERC721 token standard|

References:
* https://github.com/icon-project/IIPs/blob/master/IIPS/iip-3.md
* https://github.com/icon-project/IIPs/blob/master/IIPS/iip-2.md
* https://github.com/icon2infiniti/Samples/blob/master/IRC3/sample_irc3/sample_irc3.py

---

## Implementation
* [Idol Token](https://github.com/ibriz/ibriz-icon-foundation-idoltoken/blob/contract_update/idol_token/idol_token.py)