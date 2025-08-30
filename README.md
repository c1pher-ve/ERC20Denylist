# ERC20Denylist
This repository provides an extended implementation of the standard ERC-20 token, built upon OpenZeppelin's upgradeable contracts. The key feature of this token is an integrated denylist mechanism, which allows authorized administrators to freeze addresses, preventing them from transferring tokens or participating in any token-related activities.

This functionality is crucial for projects that require enhanced security, compliance with regulations (e.g., sanction screening, AML/CFT), or the ability to neutralize malicious actors

## Gas-Efficient Bitmasking
Instead of using a separate ```mapping(address => bool)``` to track denylisted addresses, this implementation uses a more gas-efficient technique called bitmasking.

The denylist status is stored within the most significant bit (the 256th bit) of an account's uint256 balance itself.

BALANCE_DENY_FLAG_SET_MASK: A constant defined as uint256(1) << 255, which represents a number with only the highest bit set to 1.

MAX_ALLOWED_SUPPLY: The total supply of the token is capped at (uint256(1) << 255) - 1. This cap is a critical safeguard ensuring that a regular user balance can never naturally have its highest bit set, preventing collisions with the denylist flag.

## Denylist Management
```_denylist(address)```: An internal function to add an address to the denylist. It sets the denylist flag on the user's balance. 
This function is intended to be exposed only to authorized roles in a child contract.

```_undenylist(address)```: An internal function to remove an address from the denylist, clearing the flag.

```isDenylisted(address)```: A public view function that returns true if an address is on the denylist.

```_destroyDenylistedFunds(address)```: A powerful internal function for authorized administrators to burn all tokens held by a denylisted address. This action reduces the user's balance to zero and decreases the totalSupply accordingly. The address remains on the denylist after the funds are destroyed.

## Contract Protections
Transfer Prevention: The core ```_transfer``` logic checks if either the from or to address is denylisted. If so, the entire transaction reverts. This effectively blocks all token movements involving a denylisted party.

Upgradeable: Built using OpenZeppelin's Initializable contract, allowing for future upgrades to the logic without requiring a token migration.

