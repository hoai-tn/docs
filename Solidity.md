## Data Locations \- Storage, Memory and Calldata

**memory:**
- không lưu trên blockchain  
- khai báo ở trong function  
- khai báo độ dài
- Location: Contains function arguments and is read-only.   
- Persistence: Data in calldata is immutable and cannot be modified.   
- Cost: Reading from calldata is cheaper compared to storage.

**calldata:** is somehow similar to memory, but it's only available to external functions.

**storage:** is used to store data permanently on the blockchain.

**view:** _view_ functions don't cost any gas when they're called externally by a user.
This is because _view_ functions don't actually change anything on the blockchain

**payable:** allow send ethers  

## Optimize 

uint, uint32, uint256 is the same gas but in struct is diffrent 
```
// equal gas
uint256 a;
uint8 b;

//save gas
struct Number {
  uint256 a;
  uint8 b;
}
// high gas
struct Number {
  uint a;
  uint b;
}
```
