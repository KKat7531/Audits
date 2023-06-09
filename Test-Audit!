# # Introduction

Security review of the **T-BILLY** protocol was done by KKat7531, with a focus on the security aspects of the implementation.

# Disclaimer

A smart contract security review can never verify the complete absence of vulnerabilities. This is a time, resource and expertise bound effort where I try to find as many vulnerabilities as possible. I can not guarantee 100% security after the review or if even the review will find any problems with your smart contracts.

# Protocol Overview

Reviwed project is Lending and Borrowing protocol, it contains 6 interfaces and 4 libraries(BlooPool.sol, BPSFEED.Sol, MerkleWhitelist.sol, SwapFacility.sol).
Lenders and borrowers can deposit stablecoins and swap them for tokenized treasuries by BloomPool smart coontract
The high level functionality of SwapFacility.sol is to swap the `UNDERLYING_TOKEN` to and from the `BILL_TOKEN`. 


# Severity classification

| Severity               | Impact: High | Impact: Medium | Impact: Low |
| ---------------------- | ------------ | -------------- | ----------- |
| **Likelihood: High**   | Critical     | High           | Medium      |
| **Likelihood: Medium** | High         | Medium         | Low         |
| **Likelihood: Low**    | Medium       | Low            | Low         |

# Security Assessment Summary

The following number of issues were found, categorized by their severity:

- Critical & High: 0 issues
- Medium: 0 issues
- Low: 2 issues
- Informational: 5 issues

---

# Findings Summary

| ID     | Title                                                                              | Severity      |
| ------ | ---------------------------------------------------------------------------------- | ------------- |
| [L-01] | Solmate's SafeTransferLib doesn't check whether the ERC20 contract exists          | Low           |
| [L-02] | Critical onlyOwner privileges can cause damage to the project in privateKey exploit| Low           |
| [I-01] | @OZ 4.8.2 library can be removed as it’s not used                                  | Informational |
| [I-02] |  Use stable pragma + same compiler version over all interfaces.                    | Informational |
| [I-03] | Typos in README.md                                                                 | Informational |          
| [G-01] | x = x - y costs less gas than x -= y, same for addition                            | Informational |
| [G-02] | Use x !=0 instead of x>0 for uint types                                            | Informational |

# Detailed Findings

# [L-01] Solmate's SafeTransferLib doesn't check whether the ERC20 contract exists 

## Description

Solmate's SafeTransferLib, which is often used to interact with non-compliant/unsafe ERC20 tokens, does not check whether the ERC20 contract exists. Code will not revert in case the token doesn't exist (yet).
In our case SafeTranser is used to transfer UNDERLYING_TOKEN. And ERC20 tokens to or from the pool to make a Swap and also from the Lenders and Borrowers to make a deposits in Bloompool. 

## Recommendation
According to this usage and also this inability of Solmate's SafeTransferLib, consider verifying whether a contract, i.e. token, exists before using any SafeTransferLib functions:

Example require(token.code.length != 0, "Contract does not exist");

# [L-02] Critical onlyOwner privileges can cause damage to the project in privateKey exploit

## Description
Typically, the contract’s onlyOwner is the account that deploys the contract. As a result, the onlyOwner is able to perform certain privileged activities.
onlyOwner privileges in  SwapFacility.sol are to setPool and setSpreadPrice and there is no timelock structure in the process of using these privileges.
The onlyOwner is assumed to be an EOA, since the documents do not provide information on whether the onlyOwner will be a multisign structure.

## Recommendation

1. A timelock contract should be added to use onlyOwner privileges. In this way, users can be warned in case of a possible security weakness.
2. onlyOwner can be a Multisign wallet.

# [I-01] @OZ 4.8.2 library can be removed as it’s not used     

# [I-02] Use stable pragma + same compiler version over all interfaces.   

## Description
Contracts should be deployed with the same compiler version and flags that they
have been tested with thoroughly. Locking the pragma helps to ensure that
contracts do not accidentally get deployed using, for example, an outdated
compiler version that might introduce bugs that affect the contract system
negatively.

# [I-03] Typos in README.md 
recieving - receiving
critcial - critical

# [ G-01] x = x - y costs less gas than x -= y, same for addition    

# [ G-02] Use x !=0 instead of x>0 for uint types      







