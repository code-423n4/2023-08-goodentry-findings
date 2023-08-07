Adding address validation in constructor for extra security (https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol)


Adding a require check for the validity of _asset0 and _asset1 can provide additional security by preventing the contract from being deployed with invalid or malicious tokens. 


`constructor(ILendingPool lendingPool, ERC20 _asset0, ERC20 _asset1)  {
    require( address(lendingPool) != address(0x0), "Invalid address" );
    LENDING_POOL = lendingPool;
    ASSET_0 = _asset0 < _asset1 ? _asset0 : _asset1;
    ASSET_1 = _asset0 < _asset1 ? _asset1 : _asset0;
  }
`

Recommend adding a require statement check for `_asset0` & `_asset1`
