# Introduction

A time-boxed security review of the PunksBids protocol was done by `KKat7531`, with a focus on the security aspects of the application's implementation.


# Disclaimer

A smart contract security review can never verify the complete absence of vulnerabilities. This is a time, resource and expertise bound effort where I try to find as many vulnerabilities as possible. I can not guarantee 100% security after the review or even if the review will find any problems with your smart contracts. Subsequent security reviews, bug bounty programs and on-chain monitoring are strongly recommended.

# Protocol Overview

PunksBids is a bidding platform for CryptoPunks, allowing anybody to bid with WETH for a specific CryptoPunks or attributes. Protocol itself should ensure that there is a match between the CryptoPunk ID that is for sale and the one that user requested to buy. Based on that service which the platform offer, PunksBids collect fees from the users as a profit.

## Privileged Roles & Actors

- `Owner` - an entity that can call specific methods such as to set fee rates and withdraw fees (accumulated fees as a profit).
- `Bidder` - an entity that can make a bid for a particular asset (CryptoPunk) by providing WETH in exchange.

# Severity classification

| Severity               | Impact: High | Impact: Medium | Impact: Low |
| ---------------------- | ------------ | -------------- | ----------- |
| **Likelihood: High**   | Critical     | High           | Medium      |
| **Likelihood: Medium** | High         | Medium         | Low         |
| **Likelihood: Low**    | Medium       | Low            | Low         |

**Impact** - the technical, economic and reputation damage of a successful attack

**Likelihood** - the chance that a particular vulnerability gets discovered and exploited

**Severity** - the overall criticality of the risk

# Security Assessment Summary

### Scope

The following smart contracts were in scope of the audit:

- `src/lib/*`
- `src/PunksBids.sol`

The following number of issues were found, categorized by their severity:

- Critical & High: 0 issues
- Medium: 1 issues
- Low: 2 issues
- Informational: 4 issues
- Gas Optimizations: 3

---

# Findings Summary

| ID     | Title                                                                                                                | Severity         |
| ------ | -------------------------------------------------------------------------------------------------------------------- | --------         |
| [M-01] | Missing upper limit definition in `setFeeRate` and `setLocalFeeRate`                                                 | Medium           |
| [L-01] | Use a two-step ownership transfer approach                                                                           | Low              |
| [L-02] | Usage of `ecrecover` should be replaced with usage of OpenZeppelin's ECDSA library                                   | Low              |
| [I-01] | Use latest Solidity version with a stable pragma statement.                                                          | Informational    |
| [I-02] | Empty fallback/receive functions in `PunkBids.sol`                                                                   | Informational    |
| [I-03] | `public` functions not called by the contract should be declared `external` instead                                  | Informational    |
| [I-04] | File does not contain a `SPDX Identifier`                                                                            | Informational    |
| [G-01] | `x = x + y` costs less gas than `x += y` , same for subtraction                                                      | Gas-Optimization |
| [G-02] | Expressions for constant values such as a call to keccak256, should use `immutable` rather than `constant`.          | Gas-Optimization |
| [G-03] | Use prefix not postfix in loops                                                                                      | Gas-Optimization |

# Detailed Findings

# [M-01] Missing upper limit definition in `setFeeRate` and `setLocalFeeRate`     

## Description

As seen below, `setFeeRate`  and `setLocalFeeRate` allows the Owner to set a new fee rate to any value within the range of `uint16`

```solidity
* @dev Sets a new fee rate
     * @param _feeRate The new fee rate
     */
    function setFeeRate(uint16 _feeRate) public onlyOwner {
        feeRate = _feeRate;

/**
     * @dev Sets a new local fee rate
     * @param _localFeeRate The new fee rate
     */
    function setLocalFeeRate(uint16 _localFeeRate) public onlyOwner {
        localFeeRate = _localFeeRate;

        emit LocalFeeRateUpdated(localFeeRate);
```
This two functions are used to set the fees that owner receives, whenever a user chooses to bid with WETH for a specific `CryptoPunk` and to buy it. Having this privilege without upper limit definition implemented, `Owner` can change the fee price to a bigger amount which may be bigger than then actual price of the particular asset or bigger than the bid, that user is willing to pay for it.

## Recommendation

Consider declaring a reasonable upper limit in `setFee()`, for example 10%. This would prevent potentially unwanted behavior of the `Owner` and would increase the trust of users to the contract.

# [L-01] Use a two-step ownership transfer approach   

## Description

There are 5 functions with the `onlyOwner` modifier which shows that the owner role is an important one. Make sure to use a two-step ownership transfer approach by using `Ownable2Step` from OpenZeppelin as opposed to `Ownable` as it gives you the security of not unintentionally sending the owner role to an address you do not control.

# [L-02] Usage of `ecrecover` should be replaced with usage of `OpenZeppelin's ECDSA library`

## Description

Signature malleability is one of the potential issues with `ecrecover`. By flipping s and v it is possible to create a different signature that will amount to the same hash & signer. This is fixed in `OpenZeppelin's ECDSA library`. 

# [I-01] Use latest Solidity version with a stable pragma statement.  

Using a floating pragma `^0.8.0` statement is discouraged as code can compile to different bytecodes with different compiler versions. Use a stable pragma statement to get a deterministic bytecode. Also use more recent Solidity version to get all compiler features, bugfixes and optimizations

# [I-02] Empty fallback/receive functions in `PunkBids.sol` 

```solidity
 receive() external payable {}
 fallback() external payable {}
 ```

If the intention is for the `WETH` to be used, the function should call another function, otherwise it should revert. Having no access control on the function means that someone may send Ether to the contract, and have no way to get anything back out, which can cause a loss of funds 

# [I-03] `public` functions `PunkBids.sol` not called by the contract should be declared `external` instead 

Since a function is not called from within a contract, using `external` instead of `public` will save gas. 

# [I-04] File does not contain a `SPDX Identifier` 

`File: StringUtils.sol`



# Gas Optimization findings

# [G-01] `x = x + y` costs less gas than `x += y` , same for subtraction 

`StringUtils.sol`

# [G-02] Expressions for constant values such as a call to keccak256(), should use `immutable` rather than `constant`

In a number of places a keccak("string") expression is assigned to a `constant` variable. Due to how constant variables are implemented this results in the hash being recomputed each time that the variable is used, spending the gas necessary to perform this action.
If these variables were to be `immutable` this hash is calculated once at deploy time and then the result is saved to be used directly at runtime rather than recalculating, saving the cost of hashing.

# [G-03] Use prefix not postfix in loops 

Using a prefix increment `(++i) `instead of a postfix increment `(i++)` saves gas for each loop cycle and so can have a big gas impact when the loop executes on a large number of elements.

