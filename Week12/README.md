# Week12

## RareSkills Huff Puzzles

Here are my solutions for the Huff Puzzles. An explanation will be given when I think it is needed.

All my work can be found here: https://github.com/zigtur/RareSkills-huff-puzzles


For the opcodes, see: https://www.evm.codes/

### Return1

Solution
```solidity
#define macro MAIN() = takes(0) returns(0) {
    0x01
    0x00
    mstore

    0x20
    0x00
    return
}
```

### CallValue
Solution:
```solidity
#define macro MAIN() = takes(0) returns(0) {
    // Store calldata size in memory
    callvalue           // This will store calldatasize on stack
    0x00                // memory offset
    mstore

    0x20                // size
    0x00                // memory offset
    return

}
```

### CalldataLength
Solution:
```solidity
#define macro MAIN() = takes(0) returns(0) {
    // Store calldata size in memory
    calldatasize        // This will store calldatasize on stack
    0x00                // Memory offset
    mstore

    // Return value stored in memory
    0x20                // size
    0x00                // Memory offset
    return
}
```

### MyEtherBalance
Solution:
```solidity
#define macro MAIN() = takes(0) returns(0) {
   caller               // push caller balance on stack
   balance              // pop caller on stack and push caller's balance on stack
   0x00                 // push 0x00 (memory offset)
   mstore               // store in memory

    // Return value stored in memory
   0x20                 // size
   0x00                 // Memory offset
   return
}
```

### Add1
The selector handling is pretty simple. Take the first 4 bytes of calldata, and check if it is equal to the selector. If it is, jump to addition code. Otherwise, revert.

Solution:
```solidity
#define macro MAIN() = takes(0) returns(0) {
    // get 4 first bytes
    0x00
    calldataload
    0xe0
    shr

    // add1(uint256) selector
    __FUNC_SIG(add1)
    eq
    addition
    // if selector is add1, jump to addition
    jumpi

    // revert otherwise
    0x00
    0x00
    revert

    addition:
        // Copy first argument in memory
        0x20
        0x04
        0x00
        calldatacopy

        // Load from memory
        0x00
        mload
        // Add 1
        0x01
        add

        // Store the result in memory
        0x00
        mstore

        // Return the value
        0x20
        0x00
        return
}
```

### Multiply
The code does not work, because Huff is bugged? Investigation in EVM playground shows that the end JUMPI does jump on REVERT opcode, and not JUMPDEST. 

Solution:
```solidity
#define function multiply(uint256, uint256) payable returns(uint256)


#define macro MAIN() = takes(0) returns(0) {
    // get 4 first bytes
    0x00
    calldataload
    0xe0
    shr

    // multiply(uint256, uint256) selector
    __FUNC_SIG(multiply)
    eq
    multiplication
    // if selector is multiply, jump to multiplication
    jumpi

    // revert otherwise
    0x00
    0x00
    revert

    multiplication:
        // Copy first argument in memory
        0x04 calldataload
        dup1
        iszero
        zero_case
        jumpi
        
        // Copy second argument in memory
        0x24 calldataload
        dup1
        iszero
        zero_case
        jumpi

        // multiply (c = a * b)
        mul
        dup1

        // load first argument (a) and divide result by it
        // c / a
        0x04 calldataload
        swap1
        div

        // b == c / a ?
        0x24 calldataload
        eq

        end
        jumpi

        0x6f766572666c6f77 0x40 mstore
        0x20 0x40 revert

    // push zero before end
    zero_case:
        0x00

    end:
        // store result and return
        0x40
        mstore

        0x20
        0x40
        return
}
```

### NonPayable
Solution:
```solidity
#define macro MAIN() = takes(0) returns(0) {
    0x00
    0x00
    callvalue
    iszero
    novalue jumpi
    revert
    novalue:
        return
}
```

### FooBar
Solution:
```solidity
 #define function foo() payable returns(uint256)
 #define function bar() payable returns(uint256)


#define macro MAIN() = takes(0) returns(0) {
    // get 4 first bytes
    0x00
    calldataload
    0xe0
    shr

    // foo()
    dup1
    __FUNC_SIG(foo) eq
    foo jumpi
    // bar()
    __FUNC_SIG(bar) eq
    bar jumpi
    0x00 0x00 revert

    foo:
        pop
        0x02 0x00 mstore
        retvalue jump

    bar:
        0x03 0x00 mstore
        retvalue jump

    retvalue:
        0x20 0x00 return
}
```

### SimpleStore
Solution:
```solidity
#define function store(uint256) payable returns()
#define function read() payable returns(uint256)

#define macro MAIN() = takes(0) returns(0) {
    // get 4 first bytes
    0x00
    calldataload
    0xe0
    shr

    // read()
    dup1
    __FUNC_SIG(read) eq
    read jumpi
    // store()
    __FUNC_SIG(store) eq
    store jumpi
    0x00 0x00 revert

    read:
        pop
        0x00 sload
        0x00 mstore
        retvalue jump

    store:
        0x04 calldataload
        0x00 sstore
        retvalue jump

    retvalue:
        0x20 0x00 return
}
```


### RevertCustom
Solution:
```solidity
#define error OnlyHuff()

#define macro MAIN() = takes(0) returns(0) {
    __ERROR(OnlyHuff) 0x00 mstore
    0x20 0x00 revert
}
```


### RevertString
Solution:
```solidity
#define macro MAIN() = takes(0) returns(0) {
    0x4f6e6c7920487566660000000000000000000000000000000000000000000000
    0x00 mstore
    0x09 0x00 revert
}
```


### SumArray
Solution:
```solidity
#define function sumArray(uint256[]) payable returns(uint256)


#define macro MAIN() = takes(0) returns(0) {
    // get 4 first bytes
    0x00
    calldataload
    0xe0
    shr

    // multiply(uint256, uint256) selector
    __FUNC_SIG(sumArray)
    eq
    init jumpi
    0x00 0x00 revert

    init:
        // i=0
        0x00
        // get array size
        0x04 calldataload
        0x04 add
        dup1 // save index in calldata
        calldataload
        swap1
        // sum variable
        0x00

    sum:
        dup3
        dup5
        lt
        iszero
        end jumpi

        swap1
        0x20 add
        swap1
        dup2
        calldataload
        add

        swap3
        0x01 add
        swap3
        sum jump        

    end:
        0x00 mstore
        0x20 0x00 return
}
```


### Keccak
Solution:
```solidity
#define macro MAIN() = takes(0) returns(0) {
    // copy calldata in memory
    calldatasize 0x00 0x20 calldatacopy
    // hash calldata value stored in memory
    calldatasize 0x20 sha3
    //store keccak-256 hash in memory
    0x00 mstore
    0x20 0x00 return
}
```


### MaxOfArray
Solution:
```solidity
#define function maxOfArray(uint256[]) payable returns(uint256)

#define macro MAIN() = takes(0) returns(0) {
       // get 4 first bytes
    0x00
    calldataload
    0xe0
    shr

    // maxOfArray(uint256[])
    __FUNC_SIG(maxOfArray)
    eq
    init jumpi
    0x00 0x00 revert

    init:
        // i=0
        0x00
        // get array size
        0x04 calldataload
        0x04 add
        dup1 // save index in calldata
        calldataload // get size
        dup1
        iszero
        errorRevert jumpi
        swap1
        // sum variable
        0x00

    sum:
        dup3
        dup5
        lt
        iszero
        end jumpi

        swap1
        0x20 add
        swap1
        dup1
        dup3
        calldataload
        
        gt
        greater jumpi

    increment:
        swap3
        0x01 add
        swap3
        sum jump

    greater:
        pop
        dup1
        calldataload
        increment jump

    end:
        0x00 mstore
        0x20 0x00 return

    errorRevert:
        0x20 0x00 revert
}
```

### Donations
Solution:
```solidity
#define function donated(address) payable returns(uint256)

#define macro MAIN() = takes(0) returns(0) {
    // get 4 first bytes
    0x00
    calldataload
    0xe0
    shr

    // donated(address)
    __FUNC_SIG(donated)
    eq
    donated jumpi
    // receive default function
    // preparing hash caller
    caller 0x00 mstore
    0x14 0x00 sha3

    dup1
    sload
    callvalue add
    swap1
    sstore
    0x00 0x00 return

    donated:
        // copy address in memory
        0x14 //size of an address
        0x04 // calldata offset
        0x00 // memory offset (destination)
        calldatacopy

        // load value stored in storage
        0x14 0x00 sha3
        sload
        // store it in memory and return        
        0x00
        mstore
        0x20 0x00 return
}
```

### BasicBank
Solution:
```solidity
#define function balanceOf(address) payable returns(uint256)
#define function withdraw(uint256) payable returns()

#define macro MAIN() = takes(0) returns(0) {
        // get 4 first bytes
        0x00
        calldataload
        0xe0
        shr

        // balanceOf(address)
        dup1
        __FUNC_SIG(balanceOf)
        eq
        balanceOf jumpi

        // balanceOf(address)
        __FUNC_SIG(withdraw)
        eq
        withdraw jumpi

        // receive/deposit default function
        caller
        0x00 mstore
        // start at byte 12 (0x0c) to get only address
        0x14 0x0c sha3

        dup1
        sload
        callvalue add
        swap1
        sstore
        0x00 0x00 return

    balanceOf:
        pop
        // copy address in memory
        0x14 //size of an address
        0x10 // calldata offset
        0x00 // memory offset (destination)
        calldatacopy

        // load value stored in storage
        0x14 0x00 sha3
        sload
        // store it in memory and return        
        0x00
        mstore
        0x20 0x00 return
    
    withdraw:
        // hash caller
        caller 0x00 mstore
        0x14 0x0c sha3
        dup1

        // load old value and sub the amount
        sload
        dup1
        0x04 calldataload
        swap1 sub
        // restructure stack to verify substraction
        dup1 swap2
        lt 
        errorRevert jumpi

        // store new value
        swap1
        sstore

        // send value to caller
        0x00 //retSize
        0x00 //retOffset
        0x00 //argsSize
        0x00 //argsOffset
        0x04 calldataload //value to withdraw
        caller //address
        gas //gas
        call
        // Check result
        iszero
        errorRevert jumpi
        
        0x00 0x00 return

    errorRevert:
        0x00 0x00 revert
}
```

### SimulateArray
Solution:
```solidity

```

### Emitter
Solution:
```solidity

```

### Create
Solution:
```solidity

```

### SendEther
Solution:
```solidity
#define function sendEther(address) payable returns()


#define macro MAIN() = takes(0) returns(0) {
        // get 4 first bytes
    0x00
    calldataload
    0xe0
    shr

    // read()
    __FUNC_SIG(sendEther) eq
    sendLabel jumpi
    0x00 0x00 revert

    sendLabel:
        0x00 //retSize
        0x00 //retOffset
        0x00 //argsSize
        0x00 //argsOffset
        callvalue
        0x04 calldataload
        gas
        call
}
```

### Distribute
Solution:
```solidity

```