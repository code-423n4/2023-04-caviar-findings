qa
1. On  [execute](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L460) function it after first if statment the function reverts anyway. so insted of writing if and else directly revert it.
```git
function execute(address target, bytes memory data) public payable onlyOwner returns (bytes memory) {

// call the target with the value and data

(bool success, bytes memory returnData) = target.call{value: msg.value}(data);

  

// if the call succeeded return the return data

if (success) return returnData;

  

// if we got an error bubble up the error message
++ revert();
--if (returnData.length > 0) {

// solhint-disable-next-line no-inline-assembly

--assembly {

--let returnData_size := mload(returnData)

--revert(add(32, returnData), returnData_size)

--}

--} else {

--revert();

--}

}
```

