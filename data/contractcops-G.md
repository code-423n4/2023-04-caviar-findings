## [G-01] Better variable layout in Message struct
We are able to re-position the current Message struct variables in a way which packs them in a more memory-efficient way:
```
    // Current structure in the project
    struct MessageBad {
        bytes32 id;
        bytes payload;
        uint256 timestamp;
        bytes signature;
    }

    // Modified struct for better memory packing
    struct MessageGood {
        bytes32 id;
        uint256 timestamp;
        bytes payload;
        bytes signature;
    }

    // ---------------- Testing functions ------------------
    // Uses ~22888 gas
    function TestGood() external returns (MessageGood memory){
        MessageGood memory messageGood;
        messageGood.id = 0x0;
        messageGood.timestamp = 1;
        messageGood.payload = "1";
        messageGood.payload = "1";
        return messageGood;
    }
    // Uses ~22910 gas
    function TestBad() external returns (MessageBad memory){
        MessageBad memory messageBad;
        messageBad.id = 0x0;
        messageBad.timestamp = 1;
        messageBad.payload = "1";
        messageBad.payload = "1";
        return messageBad;
    }
```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/interfaces/IStolenNftOracle.sol#L6-L13