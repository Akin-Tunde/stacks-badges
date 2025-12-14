## README.md

# LW3 Stacks Badges

This repository contains the Clarity smart contract for the LearnWeb3 (LW3) Stacks Badges, a non-transferable NFT designed to certify course completion and achievements.

The project is built using the Clarinet framework for smart contract development and uses Vitest and the `@hirosystems/clarinet-sdk` for unit testing.

## Features

*   **Non-Transferable NFT:** Badges cannot be transferred after minting, ensuring they permanently stay with the recipient's wallet.
*   **Owner-Only Minting:** Only the designated contract owner can mint new tokens, suitable for a centralized badge issuing authority.
*   **Batch Minting:** Includes a function to efficiently mint up to 1000 tokens in a single transaction.
*   **Contract Ownership Management:** The owner can transfer the `contract-owner` role to a new principal.
*   **Token URI:** Follows the standard NFT trait for metadata, combining a base URI with the token ID.

## Prerequisites

To build and test this project, you will need:

*   **Clarinet:** The command-line tool for developing and testing Stacks smart contracts.
*   **Node.js & npm:** For running the TypeScript-based unit tests.

## Setup & Installation

1.  **Install Node Dependencies:**

    ```bash
    npm install
    ```

## Contract Details (`contracts/lw3-badges.clar`)

The contract is deployed as `lw3-badges` by the `deployer` address (`ST1PQHQKV0RJXZFY1DGX8MNSNYVE3VGZJSRTPGZGM`) on the Simnet.

### Data Variables

| Variable | Type | Initial Value | Description |
| :--- | :--- | :--- | :--- |
| `contract-owner` | `principal` | `tx-sender` (Deployer) | The address authorized to mint badges and transfer ownership. |
| `last-token-id` | `uint` | `u0` | Tracks the total number of minted tokens. |

### Public Functions

| Function | Signature | Access Control | Description |
| :--- | :--- | :--- | :--- |
| `mint` | `(principal)` | `contract-owner` | Mints a single new badge to the recipient. |
| `batch-mint` | `((list 1000 principal))` | `contract-owner` | Mints badges to a list of recipients (max 1000). |
| `transfer` | `(uint principal principal)` | Always fails | Implements the NFT trait, but always returns `err-non-transferable` (`u102`). |
| `set-contract-owner`| `(principal)` | `contract-owner` | Transfers the owner role to a new address. |

### Read-Only Functions

| Function | Signature | Description |
| :--- | :--- | :--- |
| `get-last-token-id` | `()` | Returns the ID of the most recently minted token. |
| `get-token-uri` | `(uint)` | Returns the metadata URI: `base-uri` + token ID. |
| `get-owner` | `(uint)` | Returns the principal address that owns the specified token ID. |

## ⚠️ Critical Bug in `get-token-uri` ⚠️

**Note:** The current implementation of `get-token-uri` uses `(int-to-ascii token-id)`, which incorrectly converts the integer to a single ASCII character (e.g., `u1` becomes the SOH character, not the string `"1"`).

To correctly format the URI, this function **must** be updated to use the `uint-to-string` built-in function, as shown below:

```clarity
;; Original (BUGGED)
(define-read-only (get-token-uri (token-id uint)) 
    (ok (some (concat base-uri (int-to-ascii token-id))))
)

;; Corrected Implementation
(define-read-only (get-token-uri (token-id uint)) 
    (ok (some (concat base-uri (uint-to-string token-id))))
)
```

## Testing

Unit tests are written in TypeScript using Vitest and interact with the contract via the Clarinet Simnet. The tests cover initialization, all public function access controls, successful minting, non-transferability, and URI formatting.

### Run Tests

To execute the unit tests and generate coverage/cost reports:

```bash
npm run test:report
```

### Test Watch Mode

To run tests automatically whenever a contract or test file is saved:

```bash
npm run test:watch
```
