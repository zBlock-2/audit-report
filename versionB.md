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


### High: [The inclusion proof reveals whether a user holds a specific token](https://github.com/zBlock-2/summa-solvency/issues/21)
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

### High: [Wrong verifying key contract permutation length can be considered valid by validateVKPermutationsLength](https://github.com/zBlock-2/summa-solvency/issues/10)
By: **obatirou**

The function `validateVKPermutationsLength` is used to validate the number of permutations in the verifying key contract corresponds to the circuit used to generate proof for the number of cryptocurrencies the custodian committed to. The Summa contract assumes that permutations commitments begins at bytes `0x2e0` in the `vKContract` ([source](https://github.com/zBlock-2/summa-solvency/blob/fec83a747ead213261aecfaf4a01b43fff9731ee/contracts/src/Summa.sol#L151))
This can lead to wrongfully consider valid vkContracts with different permutation commitments from the circuit used by the custodian. A simple case would be a verifying contract with 2 fixed commitments but one permutation commitment less than the one being used. In this case, all others things being equals, the permutations commitments would not begin at bytes `0x2e0` but at `0x0320`. As the length of the bytes of the contracts are still equals due to one permutation in less, `validateVKPermutationsLength` would consider this vkcontract valid but in fact is not.


### High: [Fake user Inclusion Proof verified in contract](https://github.com/zBlock-2/summa-solvency/issues/9)
By: **rkdud007**

participants: @qpzm, @rkdud007

## Context

We managed to generate fake inclusion proof of a non-existing user, the user id is out of the bounds of the original CSV file. As all the balance cells’ values are zero, it might not be a critical issue in the CEX use case of proof of liability, but this becomes critical issue when Summa’s core logic is utilized in more [general application use cases](https://eprint.iacr.org/2021/1350.pdf) that inclusion itself plays an important role.

## The intended behavior of the blind/unblind advice column

`user_name` is a blind advice column, which is the original implementation of the advice column. However, `balance_i` columns are defined as an unblind advice column which is implemented in [halo2 in this commit.](https://github.com/summa-dev/halo2/commit/9b200c8436812eaa67e27d4169a33d4bc61ecd79)

As can be seen, both the blind column and unblind column share the same `region.assign_advice` method to fill in the values in the table. Unlike the blind column, unblind keeps the `&advice_values[unusable_rows_start..]` in the `Scalar::ZERO` value, which is described in [this line](https://github.com/summa-dev/halo2/blob/8386d6e64fc33baccf626869123185890b8284dc/halo2_proofs/src/plonk/prover.rs#L375). And as we use a fixed column from range_check, basically $2^{K}$ - N_USER rows of both user_name and balance_i columns are considered unusable rows.

Meaning, to visualize commitments computation to advice column polynomials step, it would look like below :

| row index | user_name | balance_1 | balance_2 | range_check |
| --- | --- | --- | --- | --- |
| 0 | alice | 10 | 100 | 0 |
| 1 | bob | 20 | 0 | 1 |
| 2 | chad | 0 | 0 | 2 |
| … | … | … | … | … |
| $2^{16} - 2$ | random value 1 | 0 | 0 | $2^{16} - 2$ |
| $2^{16} - 1$  | random value 2 | 0 | 0 | $2^{16} - 1$  |
| … | … | … | … | _ |
|  $2^{17} - 1$  | random value i | 0 | 0 | _ |

This case, it will not be problematic in checking total liability as this only verifies via the sum of balances, but for a small amount of possibility that corresponds with hash collision, there might be a change that someone asks for inclusion proof.

## Commit with KZG not using blind column

As [checked](https://github.com/summa-dev/halo2/blob/16934009f5355f9bbc6e0ae0d51bca6601ff4700/halo2_proofs/src/plonk/prover.rs#L366-L378) above, both blind/unblind columns pass an additional parameter `Blind` which blind set as a random value, and unblind set as zero.

However, if we go into the `commit_lagrange` function which is actually expected to utilize this param to compute commitment, it comments out the value and does not use it anywhere.

[Commit with KZG](https://github.com/summa-dev/halo2/blob/4455c2fff7ac742725b2aa9dd0689635c8c0dfe5/halo2_proofs/src/poly/kzg/commitment.rs#L305): 

```rust
fn commit_lagrange(
    &self,
    poly: &Polynomial<E::Scalar, LagrangeCoeff>,
    _: Blind<E::Scalar>,
) -> E::G1 {
    let mut scalars = Vec::with_capacity(poly.len());
    scalars.extend(poly.iter());
    let bases = &self.g_lagrange;
    let size = scalars.len();
    assert!(bases.len() >= size);
    best_multiexp(&scalars, &bases[0..size])
}
```

It’s different behavior from the IPA commitment version.

[Commit with IPA](https://github.com/summa-dev/halo2/blob/7a2165617195d8baa422ca7b2b364cef02380390/halo2_proofs/src/poly/ipa/commitment.rs#L88-L103) :

```jsx
/// This commits to a polynomial using its evaluations over the $2^k$ size
/// evaluation domain. The commitment will be blinded by the blinding factor
/// `r`.
fn commit_lagrange(
    &self,
    poly: &Polynomial<C::Scalar, LagrangeCoeff>,
    r: Blind<C::Scalar>,
) -> C::Curve {
    let mut tmp_scalars = Vec::with_capacity(poly.len() + 1);
    let mut tmp_bases = Vec::with_capacity(poly.len() + 1);

    tmp_scalars.extend(poly.iter());
    tmp_scalars.push(r.0);

    tmp_bases.extend(self.g_lagrange.iter());
    tmp_bases.push(self.w);

    best_multiexp::<C>(&tmp_scalars, &tmp_bases)
}
```

To sum up, we can now think this is how the table looks like: 

| row index | user_name | balance_1 | balance_2 | range_check |
| --- | --- | --- | --- | --- |
| 0 | alice | 10 | 100 | 0 |
| 1 | bob | 20 | 0 | 1 |
| 2 | chad | 0 | 0 | 2 |
| … | … | … | … | … |
| $2^{16} - 2$ | 0 | 0 | 0 | $2^{16} - 2$ |
| $2^{16} - 1$  | 0 | 0 | 0 | $2^{16} - 1$  |
| … | … | … | … | … |
|  $2^{17} - 1$  | 0 | 0 | 0 | _ |

And $2^{K}$ - N_USER index can generate fake inclusion proof of user_name, balance_i all set in 0 value. 

## PoC

We did a PoC with this [commit](https://github.com/rkdud007/summa-solvency/commit/f8075aedab730023df1799bcfc4c6241c1c0d91f) to generate a non-existing user’s inclusion proof and check if it passed the verification of InclusionVerifier.

Basically `gen_commit_and_proof.rs` binary generates [fake inclusion proof call data file](https://github.com/rkdud007/summa-solvency/commit/f8075aedab730023df1799bcfc4c6241c1c0d91f#diff-34e7c661f3dfc7f0b6910ae3e3fd6a14441bdf549dc6e105e3824699cb415df0) of user index 18. Note that csv file contains only 16 lists of users.

Run with command, [contract verifies the fake proof file](https://github.com/rkdud007/summa-solvency/commit/f8075aedab730023df1799bcfc4c6241c1c0d91f#diff-4a41339bf871e24455dfdbb09c9b15c270e69098140f30affc4e52fa745376c7) as valid :

```
❯ REPORT_GAS=true npx hardhat test    

  Summa Contract
    deployment tests
      ✓ should not deploy with invalid currencies
      ✓ should not deploy with invalid byte range
      ✓ should not deploy with invalid verification key
      ✓ should not deploy with invalid snark verifier
      ✓ should not deploy with invalid grand sum verifier
      ✓ should not deploy with invalid inclusion verifier
      ✓ should not deploy if the number of cryptocurrencies is not matching the verification key 
    verify address ownership
      ✓ should verify the address ownership and store the addresses
      ✓ should revert if the caller is not the owner
      ✓ should revert if the address ownership has already been verified
      ✓ should revert if the proof of address ownership has invalid address
      ✓ should revert if the proof of address ownership has invalid chain type
      ✓ should revert if the proof of address ownership has invalid signature
      ✓ should revert if the proof of address ownership has invalid message
      ✓ should revert if requesting proof for unverified address
    submit commitment
      ✓ should submit a valid commitment
      ✓ should revert when the grand sum proof length mismatches with the total balances
      ✓ should revert a snark proof if its length is less than the grand sum proof
      ✓ should revert due to an invalid snark proof
    verify inclusion proof
      ✓ should verify inclusion proof with `verifyInclusionProof` function
      ✓ should not verify inclusion proof with wrong snark proof
      ✓ should not verify inclusion proof with wrong challenge points
      ✓ should not verify inclusion proof with value length mismatches with config

  Verifier Contracts
    Snark Proof Verifier
      ✓ should verify snark proof
      ✓ should revert with invalid proof
    GrandSum Proof Verifier
      ✓ should verify grand sum proof
    Inclusion Proof Verifier
      ✓ should verify inclusion proof
      ✓ should revert invalid inclusion proof
```


### High: [Same username overflow as reported in version A](https://github.com/zBlock-2/summa-solvency/issues/2)
By: **bbresearcher**

**Describe the bug**
Just to surface that the same issue as on V_A still exists: https://github.com/zBlock-2/summa-solvency-diffie/issues/16



### Medium: [The return value of `GrandSumVerifier` should be tested.](https://github.com/zBlock-2/summa-solvency/issues/4)
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


### Low: [Dynamic Errors not handled in Box of Errors for Function Results](https://github.com/zBlock-2/summa-solvency/issues/8)
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


### Low: [Use of unwrap in core files and improper error handling](https://github.com/zBlock-2/summa-solvency/issues/7)
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


### Low: [Incorrect permutation length in validateVKPermuationsLength](https://github.com/zBlock-2/summa-solvency/issues/6)
By: **zeroqn**

**Describe the bug**

The error arises from the range check lookup table. For instance, when the `balanceByteRange` is set to 9, we expect to have four 2^16 lookup tables plus an additional 2^8 or 2^16 lookup table. `balanceByteRange / 2` return 4, which is wrong.


**To Reproduce**

change `balanceByteRange` to 9
https://github.com/summa-dev/summa-solvency/blob/v2/contracts/test/Summa.ts#L43


**Expected behavior**

test should fail

**Additional context**



### Low: [CSV parsing allows duplicate crypto tokens.](https://github.com/zBlock-2/summa-solvency/issues/5)
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



### Low: [The return value of `GrandSumVerifier` should be tested.](https://github.com/zBlock-2/summa-solvency/issues/4)
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

### Low: [Range check tests are unreliable regarding the number of overflows.](https://github.com/zBlock-2/summa-solvency/issues/3)
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


### Informational: [Security concerns in `Summa.sol`](https://github.com/zBlock-2/summa-solvency/issues/23)
By: **hrishibhat**

- **No timestamp validation**:
```
    function submitCommitment(
        bytes calldata snarkProof,
        bytes calldata grandSumProof,
        uint256[] memory totalBalances,
        uint256 timestamp // @audit : Future timestamp can be used. This can be used to manipulate 
```        
 `timestamp` is expected to be time at which the exchange has taken snapshot of all the balances but this timestamp is not validated. As this can be set to a future timestamp. This may lead to potential manipulations by the exchange owner by combining off-chain and on-chain processes:
- Creates inconsistencies/confusion by not maintaining a chronological order in the commitment. 
- Delaying the proof verification by promising a future commitment. 
To mitigate this add the following vallidation checks to `timestamp`
- Add a check to make sure the `timestamp` is not in the future.
- Store the last submitted timestamp and check the new timestamp is larger than the previous timestamp. 
```diff
+ uint256 public lastSubmitted;

    function submitCommitment(
        bytes calldata snarkProof,
        bytes calldata grandSumProof,
        uint256[] memory totalBalances,
        uint256 timestamp
    ) public onlyOwner {
        // Check input length
        require(totalBalances.length > 0, "Invalid total balances length");
        require(
            grandSumProof.length == (totalBalances.length * 0x40),
            "Invalid grand sum proof length"
        );
+       require(timestamp < block.timestamp, "Cannot submit future commitment");
+       require(timestamp > lastSubmitted, "Incorrect timestamp");
+       lastSubmitted = timestamp;
        ....
```

**Fixed `cryptocurrencyNames` and `cryptocurrencyChains`**:
cryptocurrencyNames and cryptocurrencyChains are currently set within the summa config inside the constructor. However this could limit flexibility if the  currencies or the chains changes over time. Allow for dynamic resizing of `cryptocurrencyNames` and `cryptocurrencyChains`. 
While this may require commitment versioning so as to not impact the previous versions of commitments when the currency count changes. 

**No validation for cryptocurrencyChains of the reported balances in a commitment**:
Commitment submissions do check if the `cryptocurrencyChains` that the reported totalBalances from the respective chains or even checked against the chains parameters within address ownership proofs within `submitProofOfAddressOwnership` . 
The following sanity check :
`require(bytes(cryptocurrencies[i].chain).length <= config.cryptocurrencyChains.length, "Cryptocurrency chain mismatch");`
 

### Informational: [Update memory locations in verifiers to save gas](https://github.com/zBlock-2/summa-solvency/issues/18)
By: **qpzm**

participants: @qpzm, @rkdud007

## Description
Verifier contracts specify memory addresses to load constants from the trusted setup.
It starts from 0x220 in [GrandSumVerifier](https://github.com/summa-dev/summa-solvency/blob/fec83a747ead213261aecfaf4a01b43fff9731ee/contracts/src/GrandSumVerifier.sol#L12) and from \0x200 in [InclusionVerifier](
https://github.com/summa-dev/summa-solvency/blob/fec83a747ead213261aecfaf4a01b43fff9731ee/contracts/src/InclusionVerifier.sol#L11).
However, it can start from 0x180.


## POC
I installed foundry to see memory and added two tests.
https://github.com/qpzm/summa-solvency/blob/v2-fix-verifier-memory/contracts/test/GrandSumVerifier.t.sol
https://github.com/qpzm/summa-solvency/blob/v2-fix-verifier-memory/contracts/test/InclusionVerifier.t.sol 

Run `forge test --debug "test_verify_grandsum_proof"` and you can see memory from 0x000 to 0x180 is used in function parameters to calculate pairing. It is the blue part.
<img width="1643" alt="image" src="https://github.com/zBlock-2/summa-solvency/assets/18223805/931b8a97-3a51-4df1-aa4e-df8426b04476">

I changed the memory location constants to start from 0x180 and compared the test results.

### GrandSumVerifier
15 gas saved.

**before**
```sh
$ forge test --match-test test_verify_grandsum_proof --gas-report

Ran 1 test for test/GrandSumVerifier.t.sol:GrandSumVerifierTest
[PASS] test_verify_grandsum_proof() (gas: 285231)
Suite result: ok. 1 passed; 0 failed; 0 skipped; finished in 13.47ms (8.14ms CPU time)
| src/GrandSumVerifier.sol:GrandSumVerifier contract |                 |        |        |        |         |
|----------------------------------------------------|-----------------|--------|--------|--------|---------|
| Deployment Cost                                    | Deployment Size |        |        |        |         |
| 277055                                             | 1068            |        |        |        |         |
| Function Name                                      | min             | avg    | median | max    | # calls |
| verifyProof                                        | 271155          | 271155 | 271155 | 271155 | 1       |
```

Memory expansion: 960 bytes
<img width="1644" alt="image" src="https://github.com/zBlock-2/summa-solvency/assets/18223805/11d08cda-1923-45b0-9dd7-b9a499738f8f">


**after**
```
| src/GrandSumVerifier.sol:GrandSumVerifier contract |                 |        |        |        |         |
|----------------------------------------------------|-----------------|--------|--------|--------|---------|
| Deployment Cost                                    | Deployment Size |        |        |        |         |
| 277043                                             | 1068            |        |        |        |         |
| Function Name                                      | min             | avg    | median | max    | # calls |
| verifyProof                                        | 271140          | 271140 | 271140 | 271140 | 1       |
```

Memory expansion: 800 bytes
<img width="1639" alt="image" src="https://github.com/zBlock-2/summa-solvency/assets/18223805/7579548d-96f7-4eda-9b51-b424b5127267">


### InclusionVerifier
12 gas saved.

**before**
```sh
$ forge test --match-test test_verify_inclusion_proof --gas-report

[PASS] test_verify_inclusion_proof() (gas: 384077)
Suite result: ok. 1 passed; 0 failed; 0 skipped; finished in 15.34ms (10.88ms CPU time)
| src/InclusionVerifier.sol:InclusionVerifier contract |                 |        |        |        |         |
|------------------------------------------------------|-----------------|--------|--------|--------|---------|
| Deployment Cost                                      | Deployment Size |        |        |        |         |
| 284813                                               | 1104            |        |        |        |         |
| Function Name                                        | min             | avg    | median | max    | # calls |
| verifyProof                                          | 365492          | 365492 | 365492 | 365492 | 1       |
```

Memory expansion: 896 bytes
(The source view of foundry debugger is unreliable)
<img width="1639" alt="image" src="https://github.com/zBlock-2/summa-solvency/assets/18223805/34dd5f54-110e-4cad-b79b-5b3c35f96158">

**after**
```
| src/InclusionVerifier.sol:InclusionVerifier contract |                 |        |        |        |         |
|------------------------------------------------------|-----------------|--------|--------|--------|---------|
| Deployment Cost                                      | Deployment Size |        |        |        |         |
| 284849                                               | 1104            |        |        |        |         |
| Function Name                                        | min             | avg    | median | max    | # calls |
| verifyProof                                          | 365480          | 365480 | 365480 | 365480 | 1       |
```

Memory expansion: 768 bytes
<img width="1642" alt="Pasted image 20240428141605" src="https://github.com/zBlock-2/summa-solvency/assets/18223805/77b444d0-784a-48c2-90db-a21687e9f11f">


## Recommendation
The maximum memory size used in both verifiers is 0x180, so I recommend to save constants from memory 0x180.
The saved memory is 160 and 128 bytes respectively. 

The gas cost for memory is increasing in a quadratic way.
- memory_size_word = (memory_byte_size + 31) / 32
- gas_cost = (new_mem_size_words ^ 2 // 512) + (3 * new_mem_size_words)

Reference: https://www.evm.codes/about#memoryexpansion

Therefore if we reduce 64 memory bytes, we can make `new_mem_size_words ^ 2 // 512` in memory gas cost to 0.
I will further think about the way.

1. `GrandSumVerifier` https://github.com/summa-dev/summa-solvency/blob/fec83a747ead213261aecfaf4a01b43fff9731ee/contracts/src/GrandSumVerifier.sol#L11-L24
```solidity
    // The memory location starts at 0x180 due to the maximum operation on the ec_pairing function being 0x180, marking the maximum memory location used
    uint256 internal constant             N_INV_MPTR = 0x180;
    uint256 internal constant             LHS_X_MPTR = 0x1a0;
    uint256 internal constant             LHS_Y_MPTR = 0x1c0;
    uint256 internal constant              G1_X_MPTR = 0x1e0;
    uint256 internal constant              G1_Y_MPTR = 0x200;
    uint256 internal constant            G2_X_1_MPTR = 0x220;
    uint256 internal constant            G2_X_2_MPTR = 0x240;
    uint256 internal constant            G2_Y_1_MPTR = 0x260;
    uint256 internal constant            G2_Y_2_MPTR = 0x280;
    uint256 internal constant      NEG_S_G2_X_1_MPTR = 0x2a0;
    uint256 internal constant      NEG_S_G2_X_2_MPTR = 0x2c0;
    uint256 internal constant      NEG_S_G2_Y_1_MPTR = 0x2e0;
    uint256 internal constant      NEG_S_G2_Y_2_MPTR = 0x300;
```

2. `InclusionVerifier` https://github.com/summa-dev/summa-solvency/blob/fec83a747ead213261aecfaf4a01b43fff9731ee/contracts/src/InclusionVerifier.sol#L10-L22
```solidity
    // The memory location starts at 0x180 due to the maximum operation on the ec_pairing function being 0x180.
    uint256 internal constant             LHS_X_MPTR = 0x180;
    uint256 internal constant             LHS_Y_MPTR = 0x1a0;
    uint256 internal constant              G1_X_MPTR = 0x1c0;
    uint256 internal constant              G1_Y_MPTR = 0x1e0;
    uint256 internal constant            G2_X_1_MPTR = 0x200;
    uint256 internal constant            G2_X_2_MPTR = 0x220;
    uint256 internal constant            G2_Y_1_MPTR = 0x240;
    uint256 internal constant            G2_Y_2_MPTR = 0x260;
    uint256 internal constant      NEG_S_G2_X_1_MPTR = 0x280;
    uint256 internal constant      NEG_S_G2_X_2_MPTR = 0x2a0;
    uint256 internal constant      NEG_S_G2_Y_1_MPTR = 0x2c0;
    uint256 internal constant      NEG_S_G2_Y_2_MPTR = 0x2e0;
```

### Informational: [Security vulnerability analysis for dependencies of summa-solvency package](https://github.com/zBlock-2/summa-solvency/issues/17)
By: **sachindkagrawal15**

### **_Security vulnerability analysis for dependencies of summa-solvency package_**

###  **_vulnerabilities results 4 errors and 4 warnings found as below :_**

### **_Error # 1_**
Crate:     eyre
Version:   0.6.11
Title:     Parts of Report are dropped as the wrong type during downcast
Date:      2024-03-05
ID:        RUSTSEC-2024-0021
URL:       https://rustsec.org/advisories/RUSTSEC-2024-0021
Solution:  Upgrade to >=0.6.12
Dependency tree:
eyre 0.6.11
└── ethers-contract-abigen 2.0.12
    ├── ethers-contract-derive 2.0.12
    │   └── ethers-contract 2.0.11
    │       ├── ethers-middleware 2.0.11
    │       │   └── ethers 2.0.11
    │       │       └── summa-solvency 0.1.0
    │       └── ethers 2.0.11
    └── ethers-contract 2.0.11

### **_Error # 2_**

Crate:     h2
Version:   0.3.24
Title:     Degradation of service in h2 servers with CONTINUATION Flood
Date:      2024-04-03
ID:        RUSTSEC-2024-0332
URL:       https://rustsec.org/advisories/RUSTSEC-2024-0332
Solution:  Upgrade to ^0.3.26 OR >=0.4.4
Dependency tree:
h2 0.3.24
├── reqwest 0.11.23
│   ├── ethers-providers 2.0.11
│   │   ├── ethers-middleware 2.0.11
│   │   │   └── ethers 2.0.11
│   │   │       └── summa-solvency 0.1.0
│   │   ├── ethers-contract 2.0.11
│   │   │   ├── ethers-middleware 2.0.11
│   │   │   └── ethers 2.0.11
│   │   └── ethers 2.0.11
│   ├── ethers-middleware 2.0.11
│   └── ethers-etherscan 2.0.12
│       └── ethers 2.0.11
└── hyper 0.14.28
    └── reqwest 0.11.23

### **_Error # 3_**

Crate:     mio
Version:   0.8.10
Title:     Tokens for named pipes may be delivered after deregistration
Date:      2024-03-04
ID:        RUSTSEC-2024-0019
URL:       https://rustsec.org/advisories/RUSTSEC-2024-0019
Solution:  Upgrade to >=0.8.11
Dependency tree:
mio 0.8.10
└── tokio 1.35.1
    ├── tokio-util 0.7.10
    │   └── h2 0.3.24
    │       ├── reqwest 0.11.23
    │       │   ├── ethers-providers 2.0.11
    │       │   │   ├── ethers-middleware 2.0.11
    │       │   │   │   └── ethers 2.0.11
    │       │   │   │       └── summa-solvency 0.1.0
    │       │   │   ├── ethers-contract 2.0.11
    │       │   │   │   ├── ethers-middleware 2.0.11
    │       │   │   │   └── ethers 2.0.11
    │       │   │   └── ethers 2.0.11
    │       │   ├── ethers-middleware 2.0.11
    │       │   └── ethers-etherscan 2.0.12
    │       │       └── ethers 2.0.11
    │       └── hyper 0.14.28
    │           └── reqwest 0.11.23
    ├── reqwest 0.11.23
    ├── hyper 0.14.28
    ├── h2 0.3.24
    ├── ethers-solc 2.0.12
    │   ├── ethers-etherscan 2.0.12
    │   └── ethers 2.0.11
    ├── ethers-providers 2.0.11
    └── ethers-middleware 2.0.11

### **_Error # 4_**

Crate:     shlex
Version:   1.2.0
Title:     Multiple issues involving quote API
Date:      2024-01-21
ID:        RUSTSEC-2024-0006
URL:       https://rustsec.org/advisories/RUSTSEC-2024-0006
Solution:  Upgrade to >=1.3.0
Dependency tree:
shlex 1.2.0
└── bindgen 0.66.1
    └── c-kzg 0.1.1
        ├── revm-primitives 1.3.0
        │   ├── revm-precompile 2.2.0
        │   │   └── revm 3.5.0
        │   │       └── halo2_solidity_verifier 0.1.0
        │   │           └── summa-solvency 0.1.0
        │   └── revm-interpreter 1.3.0
        │       └── revm 3.5.0
        └── revm-precompile 2.2.0

### **_Warning # 1_**

Crate:     serde_cbor
Version:   0.11.2
Warning:   unmaintained
Title:     serde_cbor is unmaintained
Date:      2021-08-15
ID:        RUSTSEC-2021-0127
URL:       https://rustsec.org/advisories/RUSTSEC-2021-0127
Dependency tree:
serde_cbor 0.11.2
└── criterion 0.3.6
    └── summa-solvency 0.1.0

### **_Warning # 2_**

Crate:     atty
Version:   0.2.14
Warning:   unsound
Title:     Potential unaligned read
Date:      2021-07-04
ID:        RUSTSEC-2021-0145
URL:       https://rustsec.org/advisories/RUSTSEC-2021-0145
Dependency tree:
atty 0.2.14
└── criterion 0.3.6
    └── summa-solvency 0.1.0

### **_Warning # 3_**

Crate:     const-cstr
Version:   0.3.0
Warning:   unsound
Title:     const-cstr is Unmaintained
Date:      2023-03-12
ID:        RUSTSEC-2023-0020
URL:       https://rustsec.org/advisories/RUSTSEC-2023-0020
Dependency tree:
const-cstr 0.3.0
└── yeslogic-fontconfig-sys 3.2.0
    └── font-kit 0.11.0
        └── plotters 0.3.5
            ├── summa-solvency 0.1.0
            ├── halo2_proofs 0.2.0
            │   ├── summa-solvency 0.1.0
            │   ├── halo2_solidity_verifier 0.1.0
            │   │   └── summa-solvency 0.1.0
            │   └── halo2_gadgets 0.2.0
            │       └── summa-solvency 0.1.0
            └── criterion 0.3.6
                └── summa-solvency 0.1.0

### **_Warning # 4_**

Crate:     iana-time-zone
Version:   0.1.59
Warning:   yanked
Dependency tree:
iana-time-zone 0.1.59
└── chrono 0.4.31
    ├── plotters 0.3.5
    │   ├── summa-solvency 0.1.0
    │   ├── halo2_proofs 0.2.0
    │   │   ├── summa-solvency 0.1.0
    │   │   ├── halo2_solidity_verifier 0.1.0
    │   │   │   └── summa-solvency 0.1.0
    │   │   └── halo2_gadgets 0.2.0
    │   │       └── summa-solvency 0.1.0
    │   └── criterion 0.3.6
    │       └── summa-solvency 0.1.0
    ├── ethers-etherscan 2.0.12
    │   └── ethers 2.0.11
    │       └── summa-solvency 0.1.0
    └── ethers-core 2.0.12
        ├── ethers-solc 2.0.12
        │   ├── ethers-etherscan 2.0.12
        │   └── ethers 2.0.11
        ├── ethers-signers 2.0.11
        │   ├── ethers-middleware 2.0.11
        │   │   └── ethers 2.0.11
        │   └── ethers 2.0.11
        ├── ethers-providers 2.0.11
        │   ├── ethers-middleware 2.0.11
        │   ├── ethers-contract 2.0.11
        │   │   ├── ethers-middleware 2.0.11
        │   │   └── ethers 2.0.11
        │   └── ethers 2.0.11
        ├── ethers-middleware 2.0.11
        ├── ethers-etherscan 2.0.12
        ├── ethers-contract-derive 2.0.12
        │   └── ethers-contract 2.0.11
        ├── ethers-contract-abigen 2.0.12
        │   ├── ethers-contract-derive 2.0.12
        │   └── ethers-contract 2.0.11
        ├── ethers-contract 2.0.11
        ├── ethers-addressbook 2.0.12
        │   └── ethers 2.0.11
        └── ethers 2.0.11

error: 4 vulnerabilities found!
warning: 4 warnings found

### Informational: [Security vulnerability analysis for dependencies of contracts package](https://github.com/zBlock-2/summa-solvency/issues/16)
By: **sachindkagrawal15**

### **_Security vulnerability analysis for dependencies of contracts package_**

###  **_3 vulnerabilities (2 low, 1 moderate)_**

### **_vulnerability # 1_**
@openzeppelin/contracts  4.5.0 - 4.9.5
OpenZeppelin Contracts base64 encoding may read from potentially dirty memory - https://github.com/advisories/GHSA-9vx6-7xxf-x967
node_modules/@openzeppelin/contracts

### **_vulnerability # 2_**
follow-redirects  <=1.15.5
Severity: moderate
Follow Redirects improperly handles URLs in the url.parse() function - https://github.com/advisories/GHSA-jchw-25xp-jwwc
follow-redirects' Proxy-Authorization header kept across hosts - https://github.com/advisories/GHSA-cxjh-pqwp-8mfp
node_modules/follow-redirects

### **_vulnerability # 3_**
undici  <=5.28.3
Undici's fetch with integrity option is too lax when algorithm is specified but hash value is in incorrect - https://github.com/advisories/GHSA-9qxr-qj54-h672
Undici proxy-authorization header not cleared on cross-origin redirect in fetch - https://github.com/advisories/GHSA-3787-6prv-h9w3
Undici's Proxy-Authorization header not cleared on cross-origin redirect for dispatch, request, stream, pipeline - https://github.com/advisories/GHSA-m4v8-wqvr-p9f7


### Informational: [Security vulnerability analysis for dependencies of summa-backend package](https://github.com/zBlock-2/summa-solvency/issues/15)
By: **sachindkagrawal15**

### **_3 Errors and 3 warnings observed for security vulnerability analysis of dependencies of summa-backend package  as below :_**

### **_Error # 1_**
Crate:     h2
Version:   0.3.20
Title:     Degradation of service in h2 servers with CONTINUATION Flood
Date:      2024-04-03
ID:        RUSTSEC-2024-0332
URL:       https://rustsec.org/advisories/RUSTSEC-2024-0332
Solution:  Upgrade to ^0.3.26 OR >=0.4.4
Dependency tree:
h2 0.3.20
├── reqwest 0.11.18
│   ├── summa-backend 0.1.0
│   ├── ethers-providers 2.0.7
│   │   ├── ethers-middleware 2.0.7
│   │   │   └── ethers 2.0.7
│   │   │       ├── summa-solvency 0.1.0
│   │   │       │   └── summa-backend 0.1.0
│   │   │       └── summa-backend 0.1.0
│   │   ├── ethers-contract 2.0.7
│   │   │   ├── ethers-middleware 2.0.7
│   │   │   └── ethers 2.0.7
│   │   └── ethers 2.0.7
│   ├── ethers-middleware 2.0.7
│   └── ethers-etherscan 2.0.7
│       ├── ethers-middleware 2.0.7
│       └── ethers 2.0.7
└── hyper 0.14.27
    ├── reqwest 0.11.18
    └── hyper-tls 0.5.0
        └── reqwest 0.11.18

### **_Error # 2_**

Crate:     h2
Version:   0.3.20
Title:     Resource exhaustion vulnerability in h2 may lead to Denial of Service (DoS)
Date:      2024-01-17
ID:        RUSTSEC-2024-0003
URL:       https://rustsec.org/advisories/RUSTSEC-2024-0003
Solution:  Upgrade to ^0.3.24 OR >=0.4.2

### **_Error # 3_**

Crate:     mio
Version:   0.8.8
Title:     Tokens for named pipes may be delivered after deregistration
Date:      2024-03-04
ID:        RUSTSEC-2024-0019
URL:       https://rustsec.org/advisories/RUSTSEC-2024-0019
Solution:  Upgrade to >=0.8.11
Dependency tree:
mio 0.8.8
└── tokio 1.29.1
    ├── tokio-util 0.7.8
    │   └── h2 0.3.20
    │       ├── reqwest 0.11.18
    │       │   ├── summa-backend 0.1.0
    │       │   ├── ethers-providers 2.0.7
    │       │   │   ├── ethers-middleware 2.0.7
    │       │   │   │   └── ethers 2.0.7
    │       │   │   │       ├── summa-solvency 0.1.0
    │       │   │   │       │   └── summa-backend 0.1.0
    │       │   │   │       └── summa-backend 0.1.0
    │       │   │   ├── ethers-contract 2.0.7
    │       │   │   │   ├── ethers-middleware 2.0.7
    │       │   │   │   └── ethers 2.0.7
    │       │   │   └── ethers 2.0.7
    │       │   ├── ethers-middleware 2.0.7
    │       │   └── ethers-etherscan 2.0.7
    │       │       ├── ethers-middleware 2.0.7
    │       │       └── ethers 2.0.7
    │       └── hyper 0.14.27
    │           ├── reqwest 0.11.18
    │           └── hyper-tls 0.5.0
    │               └── reqwest 0.11.18
    ├── tokio-native-tls 0.3.1
    │   ├── reqwest 0.11.18
    │   └── hyper-tls 0.5.0
    ├── summa-backend 0.1.0
    ├── reqwest 0.11.18
    ├── hyper-tls 0.5.0
    ├── hyper 0.14.27
    ├── h2 0.3.20
    ├── ethers-solc 2.0.7
    │   ├── ethers-etherscan 2.0.7
    │   └── ethers 2.0.7
    ├── ethers-providers 2.0.7
    └── ethers-middleware 2.0.7

### **_Warning # 1_**

Crate:     openssl
Version:   0.10.55
Warning:   unsound
Title:     `openssl` `X509StoreRef::objects` is unsound
Date:      2023-11-23
ID:        RUSTSEC-2023-0072
URL:       https://rustsec.org/advisories/RUSTSEC-2023-0072
Dependency tree:
openssl 0.10.55
└── native-tls 0.2.11
    ├── tokio-native-tls 0.3.1
    │   ├── reqwest 0.11.18
    │   │   ├── summa-backend 0.1.0
    │   │   ├── ethers-providers 2.0.7
    │   │   │   ├── ethers-middleware 2.0.7
    │   │   │   │   └── ethers 2.0.7
    │   │   │   │       ├── summa-solvency 0.1.0
    │   │   │   │       │   └── summa-backend 0.1.0
    │   │   │   │       └── summa-backend 0.1.0
    │   │   │   ├── ethers-contract 2.0.7
    │   │   │   │   ├── ethers-middleware 2.0.7
    │   │   │   │   └── ethers 2.0.7
    │   │   │   └── ethers 2.0.7
    │   │   ├── ethers-middleware 2.0.7
    │   │   └── ethers-etherscan 2.0.7
    │   │       ├── ethers-middleware 2.0.7
    │   │       └── ethers 2.0.7
    │   └── hyper-tls 0.5.0
    │       └── reqwest 0.11.18
    ├── reqwest 0.11.18
    └── hyper-tls 0.5.0

### **_Warning # 2_**

Crate:     ahash
Version:   0.8.3
Warning:   yanked
Dependency tree:
ahash 0.8.3
└── hashbrown 0.13.2
    └── revm-primitives 1.1.2
        ├── revm-precompile 2.0.3
        │   └── revm 3.3.0
        │       └── halo2_solidity_verifier 0.1.0
        │           └── summa-solvency 0.1.0
        │               └── summa-backend 0.1.0
        └── revm-interpreter 1.1.2
            └── revm 3.3.0

### **_Warning # 3_**

Crate:     elliptic-curve
Version:   0.13.5
Warning:   yanked
Dependency tree:
elliptic-curve 0.13.5
├── k256 0.13.1
│   ├── revm-precompile 2.0.3
│   │   └── revm 3.3.0
│   │       └── halo2_solidity_verifier 0.1.0
│   │           └── summa-solvency 0.1.0
│   │               └── summa-backend 0.1.0
│   ├── ethers-core 2.0.7
│   │   ├── ethers-solc 2.0.7
│   │   │   ├── ethers-etherscan 2.0.7
│   │   │   │   ├── ethers-middleware 2.0.7
│   │   │   │   │   └── ethers 2.0.7
│   │   │   │   │       ├── summa-solvency 0.1.0
│   │   │   │   │       └── summa-backend 0.1.0
│   │   │   │   └── ethers 2.0.7
│   │   │   └── ethers 2.0.7
│   │   ├── ethers-signers 2.0.7
│   │   │   ├── ethers-middleware 2.0.7
│   │   │   └── ethers 2.0.7
│   │   ├── ethers-providers 2.0.7
│   │   │   ├── ethers-middleware 2.0.7
│   │   │   ├── ethers-contract 2.0.7
│   │   │   │   ├── ethers-middleware 2.0.7
│   │   │   │   └── ethers 2.0.7
│   │   │   └── ethers 2.0.7
│   │   ├── ethers-middleware 2.0.7
│   │   ├── ethers-etherscan 2.0.7
│   │   ├── ethers-contract-derive 2.0.7
│   │   │   └── ethers-contract 2.0.7
│   │   ├── ethers-contract-abigen 2.0.7
│   │   │   ├── ethers-contract-derive 2.0.7
│   │   │   └── ethers-contract 2.0.7
│   │   ├── ethers-contract 2.0.7
│   │   ├── ethers-addressbook 2.0.7
│   │   │   └── ethers 2.0.7
│   │   └── ethers 2.0.7
│   ├── enr 0.8.1
│   │   └── ethers-providers 2.0.7
│   └── coins-bip32 0.8.3
│       ├── ethers-signers 2.0.7
│       └── coins-bip39 0.8.6
│           └── ethers-signers 2.0.7
├── ethers-signers 2.0.7
├── ethers-core 2.0.7
└── ecdsa 0.16.7
    └── k256 0.13.1

error: 3 vulnerabilities found!
warning: 3 allowed warnings found

### Informational: [Clippy static Analysis results for summa_backend package](https://github.com/zBlock-2/summa-solvency/issues/14)
By: **sachindkagrawal15**

### **_warning: this expression creates a reference which is immediately dereferenced by the compiler_**
  --> src/apis/round.rs:70:21
   |
70 |             signer: &signer,
   |                     ^^^^^^^ help: change this to: `signer`
   |
   = help: for further information visit https://rust-lang.github.io/rust-clippy/master/index.html#needless_borrow
   = note: `#[warn(clippy::needless_borrow)]` on by default

### **_warning: single-character string constant used as pattern_**
   --> src/apis/round.rs:140:50
    |
140 |         let parts: Vec<&str> = params_path.split("-").collect();
    |                                                  ^^^ help: try using a `char` instead: `'-'`
    |
    = help: for further information visit https://rust-lang.github.io/rust-clippy/master/index.html#single_char_pattern
    = note: `#[warn(clippy::single_char_pattern)]` on by default

### **_warning: useless conversion to the same type: `ethers::types::Bytes`_**
     --> src/contracts/generated/summa_contract.rs:14041:17
      |
14041 |                 SUMMA_BYTECODE.clone().into(),
      |                 ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ help: consider removing `.into()`: `SUMMA_BYTECODE.clone()`
      |
      = help: for further information visit https://rust-lang.github.io/rust-clippy/master/index.html#useless_conversion
      = note: `#[warn(clippy::useless_conversion)]` on by default

### **_warning: module has the same name as its containing module_**
     --> src/contracts/generated/inclusion_verifier.rs:12:1
      |
12    | / pub mod inclusion_verifier {
13    | |     #[rustfmt::skip]
14    | |     const __ABI: &str = "[{\"inputs\":[{\"internalType\":\"bytes\",\"name\":\"proof\",\"type\":\"bytes\",\"components\":[]},{\"internal...
15    | |     ///The parsed JSON ABI of the contract.
...     |
38882 | |     pub struct VerifyProofReturn(pub bool);
38883 | | }
      | |_^
      |
      = help: for further information visit https://rust-lang.github.io/rust-clippy/master/index.html#module_inception
      = note: `#[warn(clippy::module_inception)]` on by default

**_### warning: useless conversion to the same type: `ethers::types::Bytes`_**
     --> src/contracts/generated/inclusion_verifier.rs:38831:17
      |
38831 |                 INCLUSIONVERIFIER_BYTECODE.clone().into(),
      |                 ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ help: consider removing `.into()`: `INCLUSIONVERIFIER_BYTECODE.clone()`
      |
      = help: for further information visit https://rust-lang.github.io/rust-clippy/master/index.html#useless_conversion

### **_warning: module has the same name as its containing module_**
    --> src/contracts/mock/mock_erc20.rs:12:1
     |
12   | / pub mod mock_erc20 {
13   | |     #[rustfmt::skip]
14   | |     const __ABI: &str = "[{\"inputs\":[],\"stateMutability\":\"nonpayable\",\"type\":\"constructor\",\"outputs\":[]},{\"inputs\":[{\"int...
15   | |     ///The parsed JSON ABI of the contract.
...    |
7226 | |     pub struct TransferFromReturn(pub bool);
7227 | | }
     | |_^
     |
     = help: for further information visit https://rust-lang.github.io/rust-clippy/master/index.html#module_inception

### **_warning: useless conversion to the same type: `ethers::types::Bytes`_**
    --> src/contracts/mock/mock_erc20.rs:6333:17
     |
6333 |                 MOCKERC20_BYTECODE.clone().into(),
     |                 ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ help: consider removing `.into()`: `MOCKERC20_BYTECODE.clone()`
     |
     = help: for further information visit https://rust-lang.github.io/rust-clippy/master/index.html#useless_conversion

### Informational: [Clippy static Analysis results for summa_solvency package](https://github.com/zBlock-2/summa-solvency/issues/13)
By: **sachindkagrawal15**

### _**warning: very complex type used. Consider factoring parts into `type` definitions**_

   --> src/chips/merkle_sum_tree.rs:113:10
    |
113 |     ) -> Result<(AssignedCell<Fp, Fp>, AssignedCell<Fp, Fp>), Error> {
    |          ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    |
    = help: for further information visit https://rust-lang.github.io/rust-clippy/master/index.html#type_complexity
    = note: `#[warn(clippy::type_complexity)]` on by default

### **_warning: very complex type used. Consider factoring parts into `type` definitions_**
  --> src/chips/range/tests.rs:58:10
   |
58 |       ) -> Result<
   |  __________^
59 | |         (
60 | |             AssignedCell<Fp, Fp>,
61 | |             AssignedCell<Fp, Fp>,
...  |
64 | |         Error,
65 | |     > {
   | |_____^
   |
   = help: for further information visit https://rust-lang.github.io/rust-clippy/master/index.html#type_complexity

### **_warning: unneeded late initialization_**
   --> src/circuits/merkle_sum_tree.rs:304:13
    |
304 |             let sibling_hash: AssignedCell<Fp, Fp>; // hash of the sibling node
    |             ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    |
    = help: for further information visit https://rust-lang.github.io/rust-clippy/master/index.html#needless_late_init
    = note: `#[warn(clippy::needless_late_init)]` on by default
help: declare `sibling_hash` here
    |
308 |             let sibling_hash: AssignedCell<Fp, Fp> = if level == 0 {
    |             ++++++++++++++++++++++++++++++++++++++++
help: remove the assignments from the branches
    |
371 ~                 computed_sibling_hash
372 |             }
  ...
439 | 
440 ~                 computed_sibling_hash
    |

### _**warning: very complex type used. Consider factoring parts into `type` definitions**_
  --> src/circuits/utils.rs:41:6
   |
41 |   ) -> Result<
   |  ______^
42 | |     (
43 | |         ParamsKZG<Bn256>,
44 | |         ProvingKey<G1Affine>,
...  |
47 | |     &'static str,
48 | | > {
   | |_^
   |
   = help: for further information visit https://rust-lang.github.io/rust-clippy/master/index.html#type_complexity



## Final remarks

## Recommendations
## Tool Configuration
## Appendix
### [Dynamic Errors not handled in Box of Errors for Function Results](https://github.com/zBlock-2/summa-solvency/issues/8)
By: **sachindkagrawal15**
### [Use of unwrap in core files and improper error handling](https://github.com/zBlock-2/summa-solvency/issues/7)
By: **sachindkagrawal15**
