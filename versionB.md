# yAcademy Summa Version B Review

Auditors:
- [0xkarl98](https://github.com/0xkarl98)
- [0xpanicError](https://github.com/0xpanicError)
- [bbresearcher](https://github.com/bbresearcher)
- [flyingnobita](https://github.com/flyingnobita)
- [hrishibhat](https://github.com/hrishibhat)
- [igorline](https://github.com/igorline)
- [kiseln](https://github.com/kiseln)
- [nullity00](https://github.com/nullity00)
- [obatirou](https://github.com/obatirou)
- [qpzm](https://github.com/qpzm)
- [rkdud007](https://github.com/rkdud007)
- [sachindkagrawal15](https://github.com/sachindkagrawal15)
- [sebastiantf](https://github.com/sebastiantf)
- [teddav](https://github.com/teddav/)
- [thogiti](https://github.com/thogiti)
- [Y5Yash](https://github.com/Y5Yash)
- [yagnadeepxo](https://github.com/yagnadeepxo)
- [zeroqn](https://github.com/zeroqn)

## Table of Contents
- [Protocol Summary](#protocol-summary)
- [Protocol Goals](#protocol-goals)
- [Scope](#scope)
- [Code Evaluation Matrix](#code-evaluation-matrix)
- [Automated testing](#automated-testing)
- [Fuzz testing](#fuzz-testing)
- [Findings Explanation](#findings-explanation)
- [Findings](#findings)
- [Final remarks](#final-remarks)
- [Recommendations](#recommendations)
- [Tool Configuration](#tool-configuration)
- [Appendix](#appendix)
## Protocol Summary
## Protocol Goals
## Scope
## Code Evaluation Matrix
---
| Category                 | Mark    | Description |
| ------------------------ | ------- | ----------- |
| Access Control           | N/A | Any access control is left to the systems of dApps that integrate RLN |
| Mathematics              | Good | No heavy mathematical components were involved. The mathematics of the underlying proof system, Groth16, were not reviewed. RLN uses the latest version of its implementation (the circom compiler). |
| Complexity               | Good | The code is easy to understand and closely follows the specification |
| Libraries                | Good | Well known libraries were used whenever possible |
| Decentralization         | Good | Summa can be integrated in highly decentralizead and permissionless environments |
| Cryptography           | Good    | |
| Code stability           | Good   | The code was reviewed at a specific commit. The code did not changed during the review. Moreover, it is not likely to change significantly with the addition of features or updates |
| Documentation            | Good | |
| Monitoring               | N/A | The protocol is intended to be integrated by other systems or dApps which will be responsible for the monitoring |
| Testing and verification | Average | The protocol contains only a few tests for the circuits. It is recommended to add more tests to increase the test coverage |

## Automated testing
## - halo2-analyzer
## - polyexen
## - Rust Tools (clippy etc)
## Fuzz Testing
## Halo2 table image
## Findings Explanation

Findings are broken down into sections by their respective Impact:
 - Critical, High, Medium, Low Impact
     - These are findings that range from attacks that may cause loss of funds, a break in the soundness, zero-knowledge, completeness of the system, proof malleability, or cause any unintended consequences/actions that are outside the scope of the requirements
 - Informational
     - Findings including Recommendations and best practices

---

## Findings


### 1. High: [The inclusion proof reveals whether a user holds a specific token](https://github.com/zBlock-2/summa-solvency/issues/21)
By: **qpzm**

### Description
A sniffer can see whether a user has a specific token because balance polynomials are unblinded.
A sniffer, for example, may be an web3 api endpoint who can see a user's inputs for `verifyInclusionProof`.

### Process
1. Assume $B(w^i) = 0$ and check $e([B(s)]_1 - [0]_1, [1]_2) == e([Q_B(w^i)]_1, [s]_2 - [w^i]_2)$
2. If it is true, the user does not have the token. If It is false, the user has the token.

**How to know each terms?**
1. $[B(s)]_1$ is the commitment stored in the contract through `submitCommitment` by a prover.
3. $[Q_B(w^i)]_1$ is given by the verifier as an input for `verifyInclusionProof`.
4. The index of a user $i$ can be known by a rainbow table trick. It calculates $[s]_{2}- [w^i]_2$ for every i and compare with the challenge input given in `verifyInclusionProof`.

### 2. High: [Wrong verifying key contract permutation length can be considered valid by validateVKPermutationsLength](https://github.com/zBlock-2/summa-solvency/issues/10)
By: **obatirou**

The function `validateVKPermutationsLength` is used to validate the number of permutations in the verifying key contract corresponds to the circuit used to generate proof for the number of cryptocurrencies the custodian committed to. The Summa contract assumes that permutations commitments begins at bytes `0x2e0` in the `vKContract` ([source](https://github.com/zBlock-2/summa-solvency/blob/fec83a747ead213261aecfaf4a01b43fff9731ee/contracts/src/Summa.sol#L151))
This can lead to wrongfully consider valid vkContracts with different permutation commitments from the circuit used by the custodian. A simple case would be a verifying contract with 2 fixed commitments but one permutation commitment less than the one being used. In this case, all others things being equals, the permutations commitments would not begin at bytes `0x2e0` but at `0x0320`. As the length of the bytes of the contracts are still equals due to one permutation in less, `validateVKPermutationsLength` would consider this vkcontract valid but in fact is not.


### 3. High: [Fake user Inclusion Proof verified in contract](https://github.com/zBlock-2/summa-solvency/issues/9)
By: **rkdud007**

participants: @qpzm, @rkdud007

## Context

We managed to generate fake inclusion proof of a non-existing user, the user id is out of the bounds of the original CSV file. As all the balance cells’ values are zero, it might not be a critical issue in the CEX use case of proof of liability, but this becomes critical issue when Summa’s core logic is utilized in more [general application use cases](https://eprint.iacr.org/2021/1350.pdf) that inclusion itself plays an important role.

### 4. High: [Same username overflow as reported in version A](https://github.com/zBlock-2/summa-solvency/issues/2)
By: **bbresearcher**

The same issue as on V_A still exists: https://github.com/zBlock-2/audit-report/blob/parsely-edits/versionA.md#1-high-possible-overflow-in-username-in-big_intify_username-combined-with-calling-big_uint_to_fp-and-guarantee-usernames-stays-inside-field



### 1. Medium: [The return value of `GrandSumVerifier` should be tested.](https://github.com/zBlock-2/summa-solvency/issues/4)
By: **qpzm**

## Description
`GrandSumVerifier.verifyProof` is not a view function.
https://github.com/summa-dev/summa-solvency/blob/fec83a747ead213261aecfaf4a01b43fff9731ee/contracts/src/GrandSumVerifier.sol#L32

This is a generated type in `contracts/typechain-types/src/GrandSumVerifier.ts` when running tests.
```
verifyProof(
  vk: PromiseOrValue<string>,
  proof: PromiseOrValue<BytesLike>,
  values: PromiseOrValue<BigNumberish>[],
  overrides?: Overrides & { from?: PromiseOrValue<string> }
): Promise<ContractTransaction>;
```

It does not return the bool value that `GrandSumVerifier.verifyProof` returns so the test just checks whether the function is not reverted. However, the function does not revert when I change the input `total_balances`.
https://github.com/summa-dev/summa-solvency/blob/fec83a747ead213261aecfaf4a01b43fff9731ee/contracts/test/Verifiers.ts#L96

## POC
Change `total_balances` to random values and then run `npx hardhat test`.
https://github.com/summa-dev/summa-solvency/blob/fec83a747ead213261aecfaf4a01b43fff9731ee/prover/bin/commitment_solidity_calldata.json#L5-L6
```json
{
  "range_check_snark_proof": "...",
  "grand_sums_batch_proof": "...",
  "total_balances": [
    "0x1087f3e0000000000000",
    "0x197f3000000000000000"
  ]
}
```

In `Verifier.ts`, `GrandSum Proof Verifier > should verify grand sum proof` test passes.
https://github.com/summa-dev/summa-solvency/blob/fec83a747ead213261aecfaf4a01b43fff9731ee/contracts/test/Verifiers.ts#L96

## Fix
1. add view modifier to `GrandSumVerifier.verifyProof`
```diff
-    ) public returns (bool) {
+    ) public view returns (bool) {
```
https://github.com/summa-dev/summa-solvency/blob/fec83a747ead213261aecfaf4a01b43fff9731ee/contracts/src/GrandSumVerifier.sol#L32


2.  Test the return value.
```diff
-       expect(await grandSumVerifier.verifyProof(verifyingKey.address, proofs, totalBalances)).to.be.not.reverted;
+       expect(await grandSumVerifier.verifyProof(verifyingKey.address, proofs, totalBalances)).to.be.true;
```
https://github.com/summa-dev/summa-solvency/blob/fec83a747ead213261aecfaf4a01b43fff9731ee/contracts/test/Verifiers.ts#L96


3. check success and revert before pairing as in `InclusionVerifier.sol`.
```diff
    success := ec_add_tmp(success, lhs_x, lhs_y)
+   if iszero(success) {
+       revert(0, 0)
+   }
```
https://github.com/summa-dev/summa-solvency/blob/fec83a747ead213261aecfaf4a01b43fff9731ee/contracts/src/GrandSumVerifier.sol#L143-L144


### 1. Low: [Dynamic Errors not handled in Box of Errors for Function Results](https://github.com/zBlock-2/summa-solvency/issues/8)
By: **sachindkagrawal15**

When we use Box on function returning a result, the underlying error type is only known at runtime and not statically known.

There are instances in code where Box is used but there is no handling of dynamic errors in the function

Examples:

1) backend/src/apis/round.rs
     Function new
2) backend/src/contracts/signer.rs
     Function submit_proof_of_address_ownership
3) zk_prover/src/merkle_sum_tree/tree.js
Functions get_leaf_node_hash_preimage, get_middle_node_hash_preimage
4) zk_prover/src/merkle_sum_tree/utils/build_tree.rs
Function build_merkle_tree_from_leaves
5) zk_prover/src/merkle_sum_tree/mst.rs
Function from_csv


### 2. Low: [Use of unwrap in core files and improper error handling](https://github.com/zBlock-2/summa-solvency/issues/7)
By: **sachindkagrawal15**

Unwrap is usually a shortcut to bypass implementing proper error handling in Rust. Using Unwrap in production code can be dangerous and should be avoided as it crashes the application & hides actual bugs. So, This is highly undesirable in core files and is a Rust bad practice and proper error handling/transmission should be implemented.

Some of Core files in repo where unwrap is used:
1. backend/src/apis/round.rs
2. backend/src/apis/mod.rs
3. backend/build.rs
4. backend/src/contracts/signer.rs
5. zk_prover/benches/full_solvency_flow.rs
6.   zk_prover/src/chips/range/range_checks.rs
7.   zk_prover/src/circuits/merkle_sum_tree.rs
8.   zk_prover/src/merkle_sum_tree/node.rs
9.   zk_prover/src/merkle_sum_tree/utils/build_tree.rs
10. zk_prover/src/merkle_sum_tree/utils/operation_helpers.rs


### 3. Low: [Incorrect permutation length in validateVKPermuationsLength](https://github.com/zBlock-2/summa-solvency/issues/6)
By: **zeroqn**

**Describe the bug**

The error arises from the range check lookup table. For instance, when the `balanceByteRange` is set to 9, we expect to have four 2^16 lookup tables plus an additional 2^8 or 2^16 lookup table. `balanceByteRange / 2` return 4, which is wrong.


**To Reproduce**

change `balanceByteRange` to 9
https://github.com/summa-dev/summa-solvency/blob/v2/contracts/test/Summa.ts#L43


**Expected behavior**

test should fail

**Additional context**



### 4. Low: [CSV parsing allows duplicate crypto tokens.](https://github.com/zBlock-2/summa-solvency/issues/5)
By: **bbresearcher**

**Describe the bug**
The CSV parsing function and code allows for the CSD import to contain duplicate crypto tokens.
This was discussed and does not seem to have any security impact however it was decided to restrict columns to be unique.


**To Reproduce**
using any csv data as below still works.
```
username,balance_ETH_ETH,balance_ETH_ETH
dxGaEAii,11888,41163
MBlfbBGI,67823,18651
```

running the command `cargo run --release --example summa_solvency_flow` still generates successfully.
```
1. Ownership proofs are submitted successfully!
2. Commitment is submitted successfully!
3. Exported proof to user #1, as `user_1_proof.json`
4. Verifying the proof on contract veirifer for User #1: true
```

**Expected behavior**
Crypto token columns should be unique



### 5. Low: [The return value of `GrandSumVerifier` should be tested.](https://github.com/zBlock-2/summa-solvency/issues/4)
By: **qpzm**

## Description
`GrandSumVerifier.verifyProof` is not a view function.
https://github.com/summa-dev/summa-solvency/blob/fec83a747ead213261aecfaf4a01b43fff9731ee/contracts/src/GrandSumVerifier.sol#L32

This is a generated type in `contracts/typechain-types/src/GrandSumVerifier.ts` when running tests.
```
verifyProof(
  vk: PromiseOrValue<string>,
  proof: PromiseOrValue<BytesLike>,
  values: PromiseOrValue<BigNumberish>[],
  overrides?: Overrides & { from?: PromiseOrValue<string> }
): Promise<ContractTransaction>;
```

It does not return the bool value that `GrandSumVerifier.verifyProof` returns so the test just checks whether the function is not reverted. However, the function does not revert when I change the input `total_balances`.
https://github.com/summa-dev/summa-solvency/blob/fec83a747ead213261aecfaf4a01b43fff9731ee/contracts/test/Verifiers.ts#L96

## POC
Change `total_balances` to random values and then run `npx hardhat test`.
https://github.com/summa-dev/summa-solvency/blob/fec83a747ead213261aecfaf4a01b43fff9731ee/prover/bin/commitment_solidity_calldata.json#L5-L6
```json
{
  "range_check_snark_proof": "...",
  "grand_sums_batch_proof": "...",
  "total_balances": [
    "0x1087f3e0000000000000",
    "0x197f3000000000000000"
  ]
}
```

In `Verifier.ts`, `GrandSum Proof Verifier > should verify grand sum proof` test passes.
https://github.com/summa-dev/summa-solvency/blob/fec83a747ead213261aecfaf4a01b43fff9731ee/contracts/test/Verifiers.ts#L96

## Fix
1. add view modifier to `GrandSumVerifier.verifyProof`
```diff
-    ) public returns (bool) {
+    ) public view returns (bool) {
```
https://github.com/summa-dev/summa-solvency/blob/fec83a747ead213261aecfaf4a01b43fff9731ee/contracts/src/GrandSumVerifier.sol#L32


2.  Test the return value.
```diff
-       expect(await grandSumVerifier.verifyProof(verifyingKey.address, proofs, totalBalances)).to.be.not.reverted;
+       expect(await grandSumVerifier.verifyProof(verifyingKey.address, proofs, totalBalances)).to.be.true;
```
https://github.com/summa-dev/summa-solvency/blob/fec83a747ead213261aecfaf4a01b43fff9731ee/contracts/test/Verifiers.ts#L96


3. check success and revert before pairing as in `InclusionVerifier.sol`.
```diff
    success := ec_add_tmp(success, lhs_x, lhs_y)
+   if iszero(success) {
+       revert(0, 0)
+   }
```
https://github.com/summa-dev/summa-solvency/blob/fec83a747ead213261aecfaf4a01b43fff9731ee/contracts/src/GrandSumVerifier.sol#L143-L144

### 6. Low: [Range check tests are unreliable regarding the number of overflows.](https://github.com/zBlock-2/summa-solvency/issues/3)
By: **qpzm**

participants: qpzm, pia

## Description
This test data has two entries which exceed 2^64 -1.
1. user0 balance0
2. user1 balance1

https://github.com/summa-dev/summa-solvency/blob/fec83a747ead213261aecfaf4a01b43fff9731ee/csv/entry_16_overflow.csv

The test result, however, shows 4 balances break the range check. For example, user2 balance0 is zero, but the test error contains "Perform range check on balance 0 of user 2".
https://github.com/summa-dev/summa-solvency/blob/fec83a747ead213261aecfaf4a01b43fff9731ee/prover/src/circuits/tests.rs#L453-L484

## Cause
The range check chip has 3 constraints. 
https://github.com/summa-dev/summa-solvency/blob/v2/fec83a747ead213261aecfaf4a01b43fff9731ee/src/chips/range/range_check.rs#L63-L72

The third one constrains that `zs[3]` must be equal to the public instance column.
https://github.com/summa-dev/summa-solvency/blob/v2/fec83a747ead213261aecfaf4a01b43fff9731ee/src/circuits/univariate_grand_sum.rs#L165

In tests, it is 0.
https://github.com/summa-dev/summa-solvency/blob/fec83a747ead213261aecfaf4a01b43fff9731ee/prover/src/circuits/tests.rs#L451

In `Summa.sol`, it is also 0.
https://github.com/summa-dev/summa-solvency/blob/fec83a747ead213261aecfaf4a01b43fff9731ee/contracts/src/Summa.sol#L242

Therefore, all balances are constrained that `zs[3] == 0` through this chained constraints.
```
public instance == zs[3] in user0, balance
user0 balance0 zs[3] == user0 balance1 zs[3]
user0 balance1 zs[3] == user1 balance0 zs[3]
...
user15 balance0 zs[3] == user15 balance1 zs[3]
```

If a balance in the middle is not equal to 0, it breaks the range check of its subsequent balance. For example, in the above test case, it breaks two equations that user1 balance1 zs[3] is 1.
```rust
user1 balance0 zs[3] != user1 balance1 zs[3] // 0 != 1
user1 balance1 zs[3] != user2 balance0 zs[3] // 1 != 0
```

### Case1
Only one balance of user is out of range, but two error occurs. This is the case I mentioned before.
```
username,balance_ETH_ETH,balance_USDT_ETH
dxGaEAii,0,0
MBlfbBGI,0,18446744073709551616
lAhWlEWZ,0,0
...
```
https://github.com/rkdud007/summa-solvency/blob/permute-overflow-poc/prover/src/circuits/tests.rs#L438

These are two constraints that break.
```
user1 balance0 zs[3] != user1 balance1 zs[3] // 0 != 1
user1 balance1 zs[3] != user2 balance0 zs[3] // 1 != 0
```
https://github.com/rkdud007/summa-solvency/blob/permute-overflow-poc/prover/src/circuits/tests.rs#L454-L473

### Case2
In this test, there are two overflows: balance0 and balance1 of user1.
```
username,balance_ETH_ETH,balance_USDT_ETH
dxGaEAii,0,0
MBlfbBGI,18446744073709551616,18446744073709551616
lAhWlEWZ,0,0
...
```

The test gives two errors: balance 0 of user 1, balance 0 of user 2 because these two constraints that break.
```
user1 balance0 zs[3] != user1 balance1 zs[3] // 0 != 1
user1 balance1 zs[3] != user2 balance0 zs[3] // 1 != 0
```
https://github.com/rkdud007/summa-solvency/blob/permute-overflow-poc/prover/src/circuits/tests.rs#L477

### Case3
In this test, there are 4 overflows.
```
username,balance_ETH_ETH,balance_USDT_ETH
dxGaEAii,0,0
MBlfbBGI,0,18446744073709551616
lAhWlEWZ,18446744073709551616,18446744073709551616
nuZweYtO,18446744073709551616,0
gbdSwiuY,0,0
```
https://github.com/rkdud007/summa-solvency/blob/permute-overflow-poc/csv/entry_16_overflow_case_3.csv

However, only two overflow error comes.
- balance 0 of user 1
- balance 1 of user 3
```rust
user1 balance0 zs[3] != user1 balance1 zs[3] // 0 != 1
user1 balance1 zs[3] == user2 balance0 zs[3] // 1 == 1
user2 balance0 zs[3] == user2 balance1 zs[3] // 1 == 1
user2 balance1 zs[3] == user3 balance0 zs[3] // 1 == 1
user3 balance0 zs[3] == user3 balance1 zs[3] // 1 != 0
```
https://github.com/rkdud007/summa-solvency/blob/permute-overflow-poc/prover/src/circuits/tests.rs#L517

## Expected behavior
A prover should not expect the test errors to include the whole overflow list. Certainly the real proof verification just shows pass or fail unlike `MockProver` in tests. However, the test gives so confusing a result that we clarify.


### 1. Informational: [Security concerns in `Summa.sol`](https://github.com/zBlock-2/summa-solvency/issues/23)
By: **hrishibhat**

### 2. Informational: [Update memory locations in verifiers to save gas](https://github.com/zBlock-2/summa-solvency/issues/18)
By: **qpzm**

participants: @qpzm, @rkdud007

## Description
Verifier contracts specify memory addresses to load constants from the trusted setup.
It starts from 0x220 in [GrandSumVerifier](https://github.com/summa-dev/summa-solvency/blob/fec83a747ead213261aecfaf4a01b43fff9731ee/contracts/src/GrandSumVerifier.sol#L12) and from \0x200 in [InclusionVerifier](
https://github.com/summa-dev/summa-solvency/blob/fec83a747ead213261aecfaf4a01b43fff9731ee/contracts/src/InclusionVerifier.sol#L11).
However, it can start from 0x180.

## Recommendation
The maximum memory size used in both verifiers is 0x180, so I recommend to save constants from memory 0x180.
The saved memory is 160 and 128 bytes respectively. 

### 3. Informational: [Security vulnerability analysis for dependencies of summa-solvency package](https://github.com/zBlock-2/summa-solvency/issues/17)
By: **sachindkagrawal15**

### 4. Informational: [Security vulnerability analysis for dependencies of contracts package](https://github.com/zBlock-2/summa-solvency/issues/16)
By: **sachindkagrawal15**

### 5. Informational: [Security vulnerability analysis for dependencies of summa-backend package](https://github.com/zBlock-2/summa-solvency/issues/15)
By: **sachindkagrawal15**

### 6. Informational: [Clippy static Analysis results for summa_backend package](https://github.com/zBlock-2/summa-solvency/issues/14)
By: **sachindkagrawal15**


### 7. Informational: [Clippy static Analysis results for summa_solvency package](https://github.com/zBlock-2/summa-solvency/issues/13)
By: **sachindkagrawal15**

## Final remarks

## Recommendations
## Tool Configuration
## Appendix
### [Dynamic Errors not handled in Box of Errors for Function Results](https://github.com/zBlock-2/summa-solvency/issues/8)
By: **sachindkagrawal15**
### [Use of unwrap in core files and improper error handling](https://github.com/zBlock-2/summa-solvency/issues/7)
By: **sachindkagrawal15**
