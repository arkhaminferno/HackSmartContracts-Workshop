# HackSmartContracts-Vulnerabilities
Founded the vulnerabilities in those smart contracts made to test my WH hacking skills.

Smart Contracts Source:
https://github.com/clesaege/HackSmartContract/blob/master/contracts/SolidityHackingWorkshop.sol


# Here are the vulnerabilities:
 Since, All of these contracts didn't use safemath library by OpenZeppelin,so there might be many underflow/overflow errors on those contracts.

# Exercise 1 
    pragma solidity ^0.4.10;
    // Simple token you can buy and send.
    contract SimpleToken{
    mapping(address => uint) public balances;

    /// @dev Buy token at the price of 1ETH/token.
    function buyToken() payable {
        balances[msg.sender]+=msg.value / 1 ether;
    }
    
    /** @dev Send token.
     *  @param _recipient The recipient.
     *  @param _amount The amount to send.
     */
    function sendToken(address _recipient, uint _amount) {
        require(balances[msg.sender]!=0); // You must have some tokens.
        
        balances[msg.sender]-=_amount;
        balances[_recipient]+=_amount;
    }
    }

## Exercise 1 vulnerability
- Should not divide by  1 either because that will destroy all fractions. Do the accounting in wei/18 decimal places and no division.
- no checking that whether you have enough tokens.

# Exercise 2 
    // You can buy voting rights by sending ether to the contract.
    // You can vote for the value of your choice.
     pragma solidity^0.4.10;
     contract VoteTwoChoices{
        mapping(address => uint) public votingRights;
        mapping(address => uint) public votesCast;
       mapping(bytes32 => uint) public votesReceived;
    
    /// @dev Get 1 voting right per ETH sent.
    function buyVotingRights() payable {
        votingRights[msg.sender]+=msg.value/(1 ether);
    }
    
    /** @dev Vote with nbVotes for a proposition.
     *  @param _nbVotes The number of votes to cast.
     *  @param _proposition The proposition to vote for.
     */
    function vote(uint _nbVotes, bytes32 _proposition) {
        require(_nbVotes + votesCast[msg.sender]<=votingRights[msg.sender]); // Check you have enough voting rights.
        
        votesCast[msg.sender]+=_nbVotes;
        votesReceived[_proposition]+=_nbVotes;
    }
    }

## Exercise 2 vulnerability
- I can double vote.It doesn't subtract votes that i have already used
# Exercise 3
    // You can buy tokens.
    // The owner can set the price.
    pragma solidity ^0.4.10;
    contract BuyToken {
    mapping(address => uint) public balances;
    uint public price=1;
    address public owner=msg.sender;
    
    /** @dev Buy tokens.
     *  @param _amount The amount to buy.
     *  @param _price  The price to buy those in ETH.
     */
    function buyToken(uint _amount, uint _price) payable {
        require(_price>=price); // The price is at least the current price.
        require(_price * _amount * 1 ether <= msg.value); // You have paid at least the total price. \\line 81
        balances[msg.sender]+=_amount;
    }
    
    /** @dev Set the price, only the owner can do it.
     *  @param _price The new price.
     */
    function setPrice(uint _price) {
        require(msg.sender==owner);
        
        price=_price;
    }
}

## Exercise 3 vulnerability

- Integer overflow at line 81
- fix would be using safemath library from openzeppelin

# Exercise 4
    // Contract to store and redeem money.
     pragma solidity^0.4.10;
     contract Store {
       struct Safe {
        address owner;
        uint amount;
      }
    
    Safe[] public safes;
    
    /// @dev Store some ETH.
    function store() payable {
        safes.push(Safe({owner: msg.sender, amount: msg.value}));
    }
    
    /// @dev Take back all the amount stored.
    function take() {
        for (uint i; i<safes.length; ++i) {
            Safe safe = safes[i];
            if (safe.owner==msg.sender && safe.amount!=0) {
                msg.sender.transfer(safe.amount); //line 122
                safe.amount=0;                    //line 123
            }
        }
        
    }
}

## Exercise 4 vulnerability
 - Functions visibility not set ,can be called by other malicious contracts.
 - I am not so sure. Although this clearly fits a Block gas Limit Dos attack because in this contract it is using loops to take back all the amounts stored.
 - A hacker can deposit really small amounts of ethers so that it can have many ethers into the array and when the contract takes back all the amounts stored it may go out of gas condition and the money will be locked forever into the contract since the contract is using transfer keyword stipends fixed amount of gas for operation(2300 gas).
 


# Exercise 5
    // Count the total contribution of each user.
    // Assume that the one creating the contract contributed 1ETH.
    pragma solidity ^0.4.10;
    contract CountContribution{
    mapping(address => uint) public contribution;
    uint public totalContributions;
    address owner=msg.sender;
    
    /// @dev Constructor, count a contribution of 1 ETH to the creator.
    function CountContribution() public {
        recordContribution(owner, 1 ether);
    }
    
    /// @dev Contribute and record the contribution.
    function contribute() public payable {
        recordContribution(msg.sender, msg.value);
    }
    
    /** @dev Record a contribution. To be called by CountContribution and contribute.
     *  @param _user The user who contributed.
     *  @param _amount The amount of the contribution.
     */
    function recordContribution(address _user, uint _amount) {
        contribution[_user]+=_amount;
        totalContributions+=_amount;
    }
    
}
## Exercise 5 vulnerability
- free 1 ether for owner.
- recordcontribution should be internal.anyone can call it 
  and record any value.
  
# Exercise 6
    pragma solidity ^0.4.10;
    contract Token {
    mapping(address => uint) public balances;
    
    /// @dev Buy token at the price of 1ETH/token.
    function buyToken() payable {
        balances[msg.sender]+=msg.value / 1 ether;
    }
    
    /** @dev Send token.
     *  @param _recipient The recipient.
     *  @param _amount The amount to send.
     */
    function sendToken(address _recipient, uint _amount) {
        require(balances[msg.sender]>=_amount); // You must have some tokens.
        
        balances[msg.sender]-=_amount;
        balances[_recipient]+=_amount;
    }
    
    /** @dev Send all tokens.
     *  @param _recipient The recipient.
     */
    function sendAllTokens(address _recipient) {
        balances[_recipient]=+balances[msg.sender]; //line 199
        balances[msg.sender]=0;                     //line 200
    }
    
}

## Exercise 6 vulnerability
 - Inside buyToken() function msg.value Should not divide by  1 either because that will destroy all fractions. Do the accounting in wei/18 decimal places and no division.
 - no checking that whether you have enough tokens.
 - line 199 could be hacked using re entrancy
 -fix would be switching lines 199 and 200.
 ```sh
 balances[msg.sender]=0;
 balances[_recipient]=+balances[msg.sender];
 ```
            
 
# Exercise 7 
 
      // You can buy some object.
     // Further purchases are discounted.
    // You need to pay basePrice / (1 + objectBought), where objectBought is the number of object you previously bought.
    pragma solidity ^0.4.10;
     contract DiscountedBuy {
    uint public basePrice = 1 ether;
    mapping (address => uint) public objectBought;

    /// @dev Buy an object.
    function buy() payable {
        require(msg.value * (1 + objectBought[msg.sender]) == basePrice);
        objectBought[msg.sender]+=1;
    }
    
    /** @dev Return the price you'll need to pay.
     *  @return price The amount you need to pay in wei.
     */
    function price() constant returns(uint price) {
        return basePrice/(1 + objectBought[msg.sender]);
    }
    
}
 
 ## Exercise 7 vulnerability
  
  -price() is never used.
  
  # Exercise 8
    
    // You choose Head or Tail and send 1 ETH.
    // The next party send 1 ETH and try to guess what you chose.
    // If it succeed it gets 2 ETH, else you get 2 ETH.
     pragma solidity ^0.4.10;
     contract HeadOrTail {
      bool public chosen; // True if head/tail has been chosen.
      bool lastChoiceHead; // True if the choice is head.
      address public lastParty; // The last party who chose.
    
    /** @dev Must be sent 1 ETH.
     *  Choose head or tail to be guessed by the other player.
     *  @param _chooseHead True if head was chosen, false if tail was chosen.
     */
    function choose(bool _chooseHead) payable {
        require(!chosen);
        require(msg.value == 1 ether);
        
        chosen=true;
        lastChoiceHead=_chooseHead;
        lastParty=msg.sender;
    }
    
    
    function guess(bool _guessHead) payable {
        require(chosen);
        require(msg.value == 1 ether);
        
        if (_guessHead == lastChoiceHead)
            msg.sender.transfer(2 ether);
        else
            lastParty.transfer(2 ether);
            
        chosen=false;
    }
}
 
 ## Exercise 8 vulnerability
   - Front running/race condition attack may happen
     You can detect what was chosen by viewing the tx data so you can send the correct guess.
     However, the original party seeing you do that could submit the same guess as you but at higher gas followed by the opposite choice      also at higher gas thereby getting both mined first. You then lose your ether.

# Exercise 9
    // You can store ETH in this contract and redeem them.
    pragma solidity ^0.4.10;
     contract Vault {
    mapping(address => uint) public balances;

    /// @dev Store ETH in the contract.
    function store() payable {
        balances[msg.sender]+=msg.value;
    }
    
    /// @dev Redeem your ETH.
    function redeem() {
        msg.sender.call.value(balances[msg.sender])(); \\line 301
        balances[msg.sender]=0;                         \\line 302
    }
}

## Exercise 9 vulnerability
 - Can be Hacked using Re entrancy attack(DAO Hack that happened in 2016,3.6 millions ether stolen) on line number 301.
 - if you could somehow repeatedly invoke the function redeem() over and over again you will end up pulling the money out before its set    to  zero that same thing happened in the dao.
 - if an attacker wrote a contract that had a fallback that called into redeem, the flow would be line 301 pays attack contract, calls     redeem, line 301, over and over until the gas runs out without changing the state.
 - One fix if we switch line number 301 and 302 then hack could be prevented.
   
   ```sh 
   balances[msg.sender]=0;
   msg.sender.call.value(balances[msg.sender])();
   ```

# Exercise 10
      
     // You choose Head or Tail and send 1 ETH.
    // The next party send 1 ETH and try to guess what you chose.
    // If it succeed it gets 2 ETH, else you get 2 ETH.
     pragma solidity ^0.4.10;
     contract HeadTail {
    address public partyA;
    address public partyB;
    bytes32 public commitmentA;
    bool public chooseHeadB;
    uint public timeB;
    
    
    
    /** @dev Constructor, commit head or tail.
     *  @param _commitmentA is keccak256(chooseHead,randomNumber);
     */
    function HeadTail(bytes32 _commitmentA) payable {
        require(msg.value == 1 ether);
        
        commitmentA=_commitmentA;
        partyA=msg.sender;
    }
    
    /** @dev Guess the choice of party A.
     *  @param _chooseHead True if the guess is head, false otherwize.
     */
    function guess(bool _chooseHead) payable {
        require(msg.value == 1 ether);
        require(partyB==address(0));
        
        chooseHeadB=_chooseHead;
        timeB=now;
        partyB=msg.sender;
    }
    
    /** @dev Reveal the commited value and send ETH to the winner.
     *  @param _chooseHead True if head was chosen.
     *  @param _randomNumber The random number chosen to obfuscate the commitment.
     */
    function resolve(bool _chooseHead, uint _randomNumber) {
        require(msg.sender == partyA);
        require(keccak256(_chooseHead, _randomNumber) == commitmentA);
        require(this.balance >= 2 ether);
        
        if (_chooseHead == chooseHeadB)
            partyB.transfer(2 ether);
        else
            partyA.transfer(2 ether);
    }
    
    /** @dev Time out party A if it takes more than 1 day to reveal.
     *  Send ETH to party B.
     * */
    function timeOut() {
        require(now > timeB + 1 days);
        require(this.balance>=2 ether);
        partyB.transfer(2 ether);
    }
}

## Exercise 10 vulnerability
 - Front running attack/Race condition attack:
   An attacker can watch the transaction pool for transactions which may contain solutions to problems, modify or revoke the attacker's    permissions or change a state in a contract which is undesirable for the attacker. The attacker can then get the data from this     transaction and create a transaction of their own with a higher gasPrice and get their transaction included in a block before the original.
 - It would seem like party A can look at party B answer and then call headtail again if they lost.this would allow party A to get their money back and lock up the 1 eth in the contract.
  then B is locked to the input, so A can change there answer and drain the whole contract.

# Exercise 11
    
    // You can create coffers put money into it and withdraw it.
    pragma solidity ^0.4.10;
    contract Coffers {
    struct Coffer {uint[] slots;}
    mapping (address => Coffer) coffers;
    
    /** @dev Create coffers.
     *  @param _extraSlots The amount of slots to add to one's coffer.
     * */
    function createCoffers(uint _extraSlots) {
        Coffer coffer = coffers[msg.sender];
        require(coffer.slots.length+_extraSlots >= _extraSlots);
        coffer.slots.length += _extraSlots;
    }
    
    /** @dev Deposit money in one's coffer slot.
     *  @param _slot The slot to deposit money.
     * */
    function deposit(uint _slot) payable {
        Coffer coffer = coffers[msg.sender];
        coffer.slots[_slot] += msg.value;
    }
    
    /** @dev withdraw all of the money of  one's coffer slot.
     *  @param _slot The slot to withdraw money from.
     * */
    function withdraw(uint _slot) {
        Coffer coffer = coffers[msg.sender];
        msg.sender.transfer(coffer.slots[_slot]); \\line 415
        coffer.slots[_slot] = 0;                  \\line 416
    }
    }

## Exercise 11 vulnerability
- re entrancy hack on line number 415 ether transfers that are followed by state changes may be reentrant.)
- no safemath library used could go underflow/overflow error.
- fix would be switching line 415 and 416.

```sh 
coffer.slots[_slot] = 0;
msg.sender.transfer(coffer.slots[_slot]);
```








