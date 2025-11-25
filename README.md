# STX-DataForge

A  data-marketplace smart contract written in Clarity for the Stacks blockchain.


Purpose
- Let users list data assets, purchase them using STX, and manage listings and seller profiles.
- Store encrypted access keys (off-chain encrypted, stored on-chain as an ASCII string reference).
- Charge a platform fee and record marketplace transactions and simple seller stats.

Quick contract summary
- Key on-chain maps/vars:
  - `data-asset-listings` — listings keyed by `{ data-asset-id: uint }`.
  - `data-access-credentials` — encrypted access keys keyed by `{ data-asset-id: uint }`.
  - `marketplace-transactions` — purchases keyed by `{ asset-buyer: principal, purchased-asset-id: uint }`.
  - `marketplace-user-profiles` — seller stats keyed by `{ marketplace-user: principal }`.
  - `asset-id-counter`, `marketplace-fee-percentage`, `total-marketplace-transactions` — data vars.

Public functions (summary)
- `create-data-asset-listing (asset-price uint) (asset-description (string-ascii 256)) (asset-category (string-ascii 64)) (encrypted-access-key (string-ascii 512))`
  - Creates a new asset listing for `tx-sender` and returns the new asset id `(ok uX)`.
  - Validations: price > 0, non-empty strings and lengths, asset id unused.

- `purchase-data-asset (data-asset-id uint)`
  - Buyer pays the listed price. Platform fee is taken (from `marketplace-fee-percentage`).
  - Transfers STX to the seller and platform owner, records the transaction and updates seller stats.
  - Returns `(ok true)` on success.

- `retrieve-asset-access-key (data-asset-id uint)`
  - Returns encrypted access key only if the caller is the buyer recorded in `marketplace-transactions`.

- `update-asset-price (data-asset-id uint) (updated-price uint)`
  - Only asset owner may update price; price must be > 0.

- `deactivate-asset-listing (data-asset-id uint)`
  - Owner can mark listing inactive.

- `update-marketplace-fee (new-marketplace-fee-percentage uint)`
  - Admin-only (contract deployer). Fee limited to ≤ 100.

Read-only functions
- `get-asset-listing-details (data-asset-id uint)` — returns listing or `none`.
- `get-user-profile (marketplace-user principal)` — returns seller profile or `none`.
- `get-total-marketplace-transactions` — returns transaction count.
- `get-current-marketplace-fee` — returns fee percentage.

Error codes
- `err u100` — `error-unauthorized-owner` (admin-only or owner-only check failed).
- `err u101` — `error-listing-not-found` (missing listing or lookup failure).
- `err u102` — `error-asset-already-listed` (asset id collision).
- `err u103` — `error-insufficient-stx-balance` (payment transfer failed).
- `err u104` — `error-unauthorized-access` (unauthorized buyer/accessor).
- `err u105` — `error-invalid-asset-price` (invalid price value).
- `err u106` — `error-invalid-input` (invalid strings/ids etc.).

Examples (pseudo-commands)
- Create listing (as seller):
  - `(create-data-asset-listing u1000000 "GPS dataset" "transport" "<ENCRYPTED_KEY>")` -> `(ok u1)`

- Purchase listing (as buyer):
  - Attach STX for the `purchase-price` and call `(purchase-data-asset u1)` -> `(ok true)` on success.

- Retrieve key (as buyer who purchased):
  - `(retrieve-asset-access-key u1)` -> `(ok "<ENCRYPTED_KEY>")`

Testing & local development
- Recommended commands (run from project root):

```bash
clarinet check   # compile / static checks
clarinet test    # run clarinet tests (if configured)
npm test         # run project tests (existing tests/ folder)
```

- The project includes `tests/STX-DataForge.test.ts` — expand this to cover happy-path and negative tests:
  - Create a listing, purchase with sufficient STX, check balances and that `marketplace-transactions` stores a record.
  - Attempt purchase with insufficient funds and assert transfer fails.
  - Verify only the buyer can call `retrieve-asset-access-key`.
  - Verify only owner can call `update-asset-price` and `deactivate-asset-listing`.

Security considerations
- Always encrypt access keys off-chain before storing the encrypted string on-chain. Do not store plaintext credentials on-chain.
- `stx-transfer?` may fail; the contract uses `try!` around transfers so failures propagate as errors. Tests should assert failure modes.
- Validate input lengths and non-empty strings in client code as well as relying on contract checks.

Possible improvements
- Add read-only helpers or pagination to enumerate active listings (convenience for frontends).
- Add withdrawal logic for accumulated platform fees, or an owner-withdrawal pattern.
- Consider event/log patterns or off-chain indexing (e.g., using The Graph or a custom indexer) for storefront UX.

Where to place this file
- Save as `README.md` in the project root (this file). If you want a shorter README in the `contracts/` folder, we can add `contracts/README.md` with a minimal snippet.

Contribution
- Open an issue or PR with tests and a short description for changes.

License
- Add a license header or file to the repo if desired.
