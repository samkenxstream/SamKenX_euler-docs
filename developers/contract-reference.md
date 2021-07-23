# Euler Contract Interfaces

## IEuler

Main storage contract for the Euler system

### moduleIdToProxy

Lookup a proxy that can be used to interact with a module

    function moduleIdToProxy(uint moduleId) external view returns (address);

Parameters:

* **moduleId**: Fixed constant that refers to a module type (ie MODULEID__MARKETS)

Returns:

* An address that should be cast to the appropriate module interface, ie IEulerMarkets(moduleIdToProxy(2))

### AssetConfig

Euler-related configuration for an asset

    struct AssetConfig {
        address eTokenAddress;
        bool borrowIsolated;
        uint32 collateralFactor;
        uint32 borrowFactor;
        uint24 twapWindow;
    }






## IEulerMarkets

Activating and querying markets, and maintaining entered markets lists

### activateMarket

Create an Euler pool and associated EToken and DToken addresses.

    function activateMarket(address underlying) external returns (address);

Parameters:

* **underlying**: The address of an ERC20-compliant token. There must be an initialised uniswap3 pool for the underlying/reference asset pair.

Returns:

* The created EToken, or the existing EToken if already activated.

### activatePToken

Create a pToken and activate it on Euler. pTokens are protected wrappers around assets that prevent borrowing.

    function activatePToken(address underlying) external returns (address);

Parameters:

* **underlying**: The address of an ERC20-compliant token. There must already be an activated market on Euler for this underlying, and it must have a non-zero collateral factor.

Returns:

* The created pToken, or an existing one if already activated.

### underlyingToEToken

Given an underlying, lookup the associated EToken

    function underlyingToEToken(address underlying) external view returns (address);

Parameters:

* **underlying**: Token address

Returns:

* EToken address, or address(0) if not activated

### underlyingToDToken

Given an underlying, lookup the associated DToken

    function underlyingToDToken(address underlying) external view returns (address);

Parameters:

* **underlying**: Token address

Returns:

* DToken address, or address(0) if not activated

### underlyingToPToken

Given an underlying, lookup the associated PToken

    function underlyingToPToken(address underlying) external view returns (address);

Parameters:

* **underlying**: Token address

Returns:

* PToken address, or address(0) if it doesn't exist

### underlyingToAssetConfig

Looks up the Euler-related configuration for a token

    function underlyingToAssetConfig(address underlying) external view returns (IEuler.AssetConfig memory);

Parameters:

* **underlying**: Token address

Returns:

* Configuration struct

### eTokenToUnderlying

Given an EToken address, looks up the associated underlying

    function eTokenToUnderlying(address eToken) external view returns (address);

Parameters:

* **eToken**: EToken address

Returns:

* Token address

### eTokenToDToken

Given an EToken address, looks up the associated DToken underlying

    function eTokenToDToken(address eToken) external view returns (address);

Parameters:

* **eToken**: EToken address

Returns:

* DToken address

### interestRateModel

Looks up an asset's currently configured interest rate model

    function interestRateModel(address underlying) external view returns (uint);

Parameters:

* **underlying**: Token address

Returns:

* Module ID that represents the interest rate model (IRM)

### interestRate

Retrieves the current interest rate for an asset

    function interestRate(address underlying) external view returns (int96);

Parameters:

* **underlying**: Token address

Returns:

* The interest rate in yield-per-second, scaled by 10**27

### interestAccumulator

Retrieves the current interest rate accumulator for an asset

    function interestAccumulator(address underlying) external view returns (uint);

Parameters:

* **underlying**: Token address

Returns:

* An opaque accumulator that increases as interest is accrued

### reserveFee

Retrieves the reserve fee in effect for an asset

    function reserveFee(address underlying) external view returns (uint32);

Parameters:

* **underlying**: Token address

Returns:

* Amount of interest that is redirected to the reserves, as a fraction scaled by RESERVE_FEE_SCALE (4e9)

### getPricingConfig

Retrieves the pricing config for an asset

    function getPricingConfig(address underlying) external view returns (uint16 pricingType, uint32 pricingParameters, address pricingForwarded);

Parameters:

* **underlying**: Token address

Returns:

* **pricingType**: (1=pegged, 2=uniswap3)
* **pricingParameters**: If uniswap3 pricingType then this represents the uniswap pool fee used, otherwise unused
* **pricingForwarded**: If forwarded pricingType then this is the address prices are forwarded to, otherwise address(0)

### getEnteredMarkets

Retrieves the list of entered markets for an account (assets enabled for collateral or borrowing)

    function getEnteredMarkets(address account) external view returns (address[] memory);

Parameters:

* **account**: User account

Returns:

* List of underlying token addresses

### enterMarket

Add an asset to the entered market list, or do nothing if already entered

    function enterMarket(uint subAccountId, address newMarket) external;

Parameters:

* **subAccountId**: 0 for primary, 1-255 for a sub-account
* **newMarket**: Underlying token address



### exitMarket

Remove an asset from the entered market list, or do nothing if not already present

    function exitMarket(uint subAccountId, address oldMarket) external;

Parameters:

* **subAccountId**: 0 for primary, 1-255 for a sub-account
* **oldMarket**: Underlying token address




## IEulerExec

Batch executions, liquidity check deferrals, and interfaces to fetch prices and account liquidity

### LiquidityStatus

Liquidity status for an account, either in aggregate or for a particular asset

    struct LiquidityStatus {
        uint collateralValue;
        uint liabilityValue;
        uint numBorrows;
        bool borrowIsolated;
    }





### AssetLiquidity

Aggregate struct for reporting detailed (per-asset) liquidity for an account

    struct AssetLiquidity {
        address underlying;
        LiquidityStatus status;
    }





### EulerBatchItem

Single item in a batch request

    struct EulerBatchItem {
        bool allowError;
        address proxyAddr;
        bytes data;
    }





### EulerBatchItemResponse

Single item in a batch response

    struct EulerBatchItemResponse {
        bool success;
        bytes result;
    }





### liquidity

Compute aggregate liquidity for an account

    function liquidity(address account) external returns (LiquidityStatus memory status);

Parameters:

* **account**: User address

Returns:

* **status**: Aggregate liquidity (sum of all entered assets)

### detailedLiquidity

Compute detailed liquidity for an account, broken down by asset

    function detailedLiquidity(address account) external returns (AssetLiquidity[] memory assets);

Parameters:

* **account**: User address

Returns:

* **assets**: List of user's entered assets and each asset's corresponding liquidity

### getPrice

Retrieve Euler's view of an asset's price

    function getPrice(address underlying) external returns (uint twap, uint twapPeriod);

Parameters:

* **underlying**: Token address

Returns:

* **twap**: Time-weighted average price
* **twapPeriod**: TWAP duration, either the twapWindow value in AssetConfig, or less if that duration not available

### getPriceFull

Retrieve Euler's view of an asset's price, as well as the current marginal price on uniswap

    function getPriceFull(address underlying) external returns (uint twap, uint twapPeriod, uint currPrice);

Parameters:

* **underlying**: Token address

Returns:

* **twap**: Time-weighted average price
* **twapPeriod**: TWAP duration, either the twapWindow value in AssetConfig, or less if that duration not available
* **currPrice**: The current marginal price on uniswap3 (informational: not used anywhere in the Euler protocol)

### deferLiquidityCheck

Defer liquidity checking for an account, to perform rebalancing, flash loans, etc. msg.sender must implement IDeferredLiquidityCheck

    function deferLiquidityCheck(address account, bytes memory data) external;

Parameters:

* **account**: The account to defer liquidity for. Usually address(this), although not always
* **data**: Passed through to the onDeferredLiquidityCheck() callback, so contracts don't need to store transient data in storage



### batchDispatch

Execute several operations in a single transaction

    function batchDispatch(EulerBatchItem[] calldata items, address[] calldata deferLiquidityChecks) external returns (EulerBatchItemResponse[] memory);

Parameters:

* **items**: List of operations to execute
* **deferLiquidityChecks**: List of user accounts to defer liquidity checks for

Returns:

* List of operation results

### trackAverageLiquidity

Enable average liquidity tracking for your account. Operations will cost more gas, but you may get additional benefits when performing liquidations

    function trackAverageLiquidity(uint subAccountId) external;

Parameters:

* **subAccountId**: subAccountId 0 for primary, 1-255 for a sub-account



### unTrackAverageLiquidity

Disable average liquidity tracking for your account

    function unTrackAverageLiquidity(uint subAccountId) external;

Parameters:

* **subAccountId**: subAccountId 0 for primary, 1-255 for a sub-account



### getAverageLiquidity

Retrieve the average liquidity for an account

    function getAverageLiquidity(address account) external returns (uint);

Parameters:

* **account**: User account (xor in subAccountId, if applicable)

Returns:

* The average liquidity, in terms of the reference asset, and post risk-adjustment


## IEulerEToken

Tokenised representation of assets

### name

Pool name, ie "Euler Pool: DAI"

    function name() external view returns (string memory);





### symbol

Pool symbol, ie "eDAI"

    function symbol() external view returns (string memory);





### decimals

Decimals, always normalised to 18.

    function decimals() external pure returns (uint8);





### totalSupply

Sum of all balances, in internal book-keeping units (non-increasing)

    function totalSupply() external view returns (uint);





### totalSupplyUnderlying

Sum of all balances, in underlying units (increases as interest is earned)

    function totalSupplyUnderlying() external view returns (uint);





### balanceOf

Balance of a particular account, in internal book-keeping units (non-increasing)

    function balanceOf(address account) external view returns (uint);





### balanceOfUnderlying

Balance of a particular account, in underlying units (increases as interest is earned)

    function balanceOfUnderlying(address account) external view returns (uint);





### reserveBalance

Balance of the reserves, in internal book-keeping units (non-increasing)

    function reserveBalance() external view returns (uint);





### reserveBalanceUnderlying

Balance of the reserves, in underlying units (increases as interest is earned)

    function reserveBalanceUnderlying() external view returns (uint);





### deposit

Transfer underlying tokens from sender to the Euler pool, and increase account's eTokens

    function deposit(uint subAccountId, uint amount) external;

Parameters:

* **subAccountId**: 0 for primary, 1-255 for a sub-account
* **amount**: In underlying units (use max uint256 for full underlying token balance)



### withdraw

Transfer underlying tokens from Euler pool to sender, and decrease account's eTokens

    function withdraw(uint subAccountId, uint amount) external;

Parameters:

* **subAccountId**: 0 for primary, 1-255 for a sub-account
* **amount**: In underlying units (use max uint256 for full pool balance)



### mint

Mint eTokens and a corresponding amount of dTokens ("self-borrow")

    function mint(uint subAccountId, uint amount) external;

Parameters:

* **subAccountId**: 0 for primary, 1-255 for a sub-account
* **amount**: In underlying units



### burn

Pay off dToken liability with eTokens ("self-repay")

    function burn(uint subAccountId, uint amount) external;

Parameters:

* **subAccountId**: 0 for primary, 1-255 for a sub-account
* **amount**: In underlying units (use max uint256 to repay full dToken balance)



### approve

Allow spender to access an amount of your eTokens in sub-account 0

    function approve(address spender, uint amount) external returns (bool);

Parameters:

* **spender**: Trusted address
* **amount**: Use max uint256 for "infinite" allowance



### approveSubAccount

Allow spender to access an amount of your eTokens in a particular sub-account

    function approveSubAccount(uint subAccountId, address spender, uint amount) external returns (bool);

Parameters:

* **subAccountId**: 0 for primary, 1-255 for a sub-account
* **spender**: Trusted address
* **amount**: Use max uint256 for "infinite" allowance



### allowance

Retrieve the current allowance

    function allowance(address holder, address spender) external view returns (uint);

Parameters:

* **holder**: Xor with the desired sub-account ID (if applicable)
* **spender**: Trusted address



### transfer

Transfer eTokens to another address (from sub-account 0)

    function transfer(address to, uint amount) external returns (bool);

Parameters:

* **to**: Xor with the desired sub-account ID (if applicable)
* **amount**: In internal book-keeping units (as returned from balanceOf). Use max uint256 for full balance.



### transferFrom

Transfer eTokens from one address to another

    function transferFrom(address from, address to, uint amount) external returns (bool);

Parameters:

* **from**: This address must've approved the to address, or be a sub-account of msg.sender
* **to**: Xor with the desired sub-account ID (if applicable)
* **amount**: In internal book-keeping units (as returned from balanceOf). Use max uint256 for full balance.




## IEulerDToken

Tokenised representation of debts

### name

Debt token name, ie "Euler Debt: DAI"

    function name() external view returns (string memory);





### symbol

Debt token symbol, ie "dDAI"

    function symbol() external view returns (string memory);





### decimals

Decimals, always normalised to 18.

    function decimals() external pure returns (uint8);





### totalSupply

Sum of all outstanding debts, in underlying units (increases as interest is accrued)

    function totalSupply() external view returns (uint);





### totalSupplyExact

Sum of all outstanding debts, in underlying units with extra precision

    function totalSupplyExact() external view returns (uint);





### balanceOf

Debt owed by a particular account, in underlying units

    function balanceOf(address account) external view returns (uint);





### balanceOfExact

Debt owed by a particular account, in underlying units with extra precision

    function balanceOfExact(address account) external view returns (uint);





### borrow

Transfer underlying tokens from the Euler pool to the sender, and increase sender's dTokens

    function borrow(uint subAccountId, uint amount) external;

Parameters:

* **subAccountId**: 0 for primary, 1-255 for a sub-account
* **amount**: In underlying units (use max uint256 for all available tokens)



### repay

Transfer underlying tokens from the sender to the Euler pool, and decrease sender's dTokens

    function repay(uint subAccountId, uint amount) external;

Parameters:

* **subAccountId**: 0 for primary, 1-255 for a sub-account
* **amount**: In underlying units (use max uint256 for full debt owed)



### approve

Allow spender to send an amount of dTokens to your sub-account 0

    function approve(address spender, uint amount) external returns (bool);

Parameters:

* **spender**: Trusted address
* **amount**: Use max uint256 for "infinite" allowance



### approveSubAccount

Allow spender to send an amount of dTokens to a particular sub-account

    function approveSubAccount(uint subAccountId, address spender, uint amount) external returns (bool);

Parameters:

* **subAccountId**: 0 for primary, 1-255 for a sub-account
* **spender**: Trusted address
* **amount**: Use max uint256 for "infinite" allowance



### allowance

Retrieve the current allowance

    function allowance(address holder, address spender) external view returns (uint);

Parameters:

* **holder**: Xor with the desired sub-account ID (if applicable)
* **spender**: Trusted address



### transfer

Transfer dTokens to another address (from sub-account 0)

    function transfer(address to, uint amount) external returns (bool);

Parameters:

* **to**: Xor with the desired sub-account ID (if applicable)
* **amount**: In underlying units. Use max uint256 for full balance.



### transferFrom

Transfer dTokens from one address to another

    function transferFrom(address from, address to, uint amount) external returns (bool);

Parameters:

* **from**: Xor with the desired sub-account ID (if applicable)
* **to**: This address must've approved the from address, or be a sub-account of msg.sender
* **amount**: In underlying. Use max uint256 for full balance.




## IEulerLiquidation

Liquidate users who are in collateral violation to protect lenders

### LiquidationOpportunity

Information about a prospective liquidation opportunity

    struct LiquidationOpportunity {
        uint repay;
        uint yield;
        uint healthScore;
    
        // Only populated if repay > 0:
        uint baseDiscount;
        uint discount;
        uint conversionRate;
    }





### checkLiquidation

Checks to see if a liquidation would be profitable, without actually doing anything

    function checkLiquidation(address liquidator, address violator, address underlying, address collateral) external returns (LiquidationOpportunity memory liqOpp);

Parameters:

* **liquidator**: Address that will initiate the liquidation
* **violator**: Address that may be in collateral violation
* **underlying**: Token that is to be repayed
* **collateral**: Token that is to be seized

Returns:

* **liqOpp**: The details about the liquidation opportunity

### liquidate

Attempts to perform a liquidation

    function liquidate(address violator, address underlying, address collateral, uint repay, uint minYield) external;

Parameters:

* **violator**: Address that may be in collateral violation
* **underlying**: Token that is to be repayed
* **collateral**: Token that is to be seized
* **repay**: The amount of underlying DTokens to be transferred from violator to sender, in units of underlying
* **minYield**: The minimum acceptable amount of collateral ETokens to be transferred from violator to sender, in units of collateral




## IEulerPToken

Protected Tokens are simple wrappers for tokens, allowing you to use tokens as collateral without permitting borrowing

### wrap

Convert underlying tokens to pTokens

    function wrap(uint amount) external;

Parameters:

* **amount**: In underlying units (which are equivalent to pToken units)



### unwrap

Convert pTokens to underlying tokens

    function unwrap(uint amount) external;

Parameters:

* **amount**: In pToken units (which are equivalent to underlying units)


