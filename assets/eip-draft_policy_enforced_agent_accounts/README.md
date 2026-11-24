# Policy-Enforced Agent Accounts: reference implementation

All files are CC0-1.0.

| File | What it is |
|---|---|
| `IPolicyEnforcedAccount.sol` | The interface (ERC-165 id `0x7e0f19f9`) |
| `PolicyEnforcedRegistry.sol` | The reference registry: no owner, no upgrade path, holds no funds. The account-owned stop revision requires a new deployment; see below. |
| `PolicyDecisionReceipt.sol` | Pure helpers to verify a decision receipt on chain (EIP-712 domain version "2") |
| `PolicyEnforcedRegistry.t.sol`, `PolicyVectors.t.sol` | Foundry tests: interface, events, ERC-165, no cross-account writes, the time rules (each review point has a test that shows the first draft's behavior and the fix), fuzzing, and the test vectors |
| `v1/` | The first draft's registry, interface and receipt helpers, deployed at `0x20D6B38B47F7cb765A2318a1f318ed54a6f87e92` (interface `0xcc16d336`). Kept for the comparison tests and for verifying receipts issued under it |
| `vectors.json` | Test vectors. The receipt key is test-only and public; never fund it |
| `client/src/*.ts` | A TypeScript reader and receipt verifier (viem), published as the npm package `policy-enforced-accounts` (0.2.0) |
| `client/independent-read.mjs` | Reads an account's signers, delay, policy and pending state, fetches and checks the policy document, and verifies a receipt, using only the chain |

To run the Solidity tests, put the `.sol` files under `src/peaa/` (and `v1/` under `src/peaa/v1/`) and the tests under `test/` in a Foundry project with `forge-std`, then point `PolicyVectors.t.sol` at `vectors.json`.

## Account-owned stop revision

Once the account asserts a stop, only it can lift it. This also applies when the
publisher stopped first: the account's assertion records ownership before the
idempotent-return check and emits `EmergencyStop`. A repeated publisher stop,
signer rotation, policy change or clearing/re-declaration does not release it.
After an explicit account lift, ordinary publisher stop/resume cycles work again.
The read interface, ERC-165 ID, event signatures and receipt vectors are unchanged.

This implements the recovery gap raised by Zealynx and acknowledged by the author
in [forum posts 9 and 10](https://ethereum-magicians.org/t/30467/10).
Implementation and regression contribution: Ted (@haot3x).

The existing v2 registry at `0xCa3bAE26d29a3A2cED7B87Fa996ca4C44367448d` on chain
4663, deployed through `0x4e59b44847b379578588920cA78FbF26c0B4956C` with salt
`keccak256("policy-enforced-accounts registry v2")`, is unchanged. It does **not**
have account-owned stop protection. The revised source is not a claim that this
non-upgradeable deployment has been patched.
