# yAcademy Summa Version A Review

Auditors:
- [0xKarl98](https://github.com/0xKarl98)
- [0xpanicError](https://github.com/0xpanicError)
- [bbresearcher](https://github.com/bbresearcher)
- [flyingnobita](https://github.com/flyingnobita)
- [hrishibhat](https://github.com/hrishibhat)
- [igorline](https://github.com/igorline)
- [irnb](https://github.com/irnb)
- [kiseln](https://github.com/kiseln)
- [nullity00](https://github.com/nullity00)
- [obatirou](https://github.com/obatirou)
- [qpzm](https://github.com/qpzm)
- [rkdud007](https://github.com/rkdud007)
- [sachindkagrawal15](https://github.com/sachindkagrawal15)
- [sebastiantf](https://github.com/sebastiantf)
- [teddav](https://github.com/teddav)
- [thogiti](https://github.com/thogiti)
- [Y5Yash](https://github.com/Y5Yash)
- [yagnadeepxo](https://github.com/yagnadeepxo)
- [zeroqn](https://github.com/zeroqn)
- [zzzuhaibmohd](https://github.com/zzzuhaibmohd)


## Table of Contents
- [Protocol Summary](#protocol-summary)
- [Methodology](#methodology)
- [Scope](#scope)
- [Findings](#findings)
  - [High](#high):
    - Possible Overflow in username in big_intify_username combined with calling big_uint_to_fp
    - Sum balance overflow
    - Inconsistency in range checks
  - [Low](#low)
    - Mixed endian usage in code
  - [Informational](#informational)
    - Range check uses lookup_any instead of lookup
    - `InclusionVerifier.yul`not generated
    - Improvement to public inputs in contract
    - Use only mapping for `addressOwnershipProofs`
    - `Summa.sol` : Issue with `submitProofOfAddressOwnership()`
    - `Summa.sol` : Ownable: Does not implement 2-Step-Process for transferring ownership
    - Potential `Summa::submitCommitment()` Gas limits
    - Magic numbers used in code of MST Circuit to create PoseidonChip
    - Review of the `Summa.sol` smart contract
- [Automated testing](#automated-testing)
- [Final remarks](#final-remarks)
- [Recommendations](#recommendations)
- [Tool Configuration](#tool-configuration)
- [Appendix](#appendix)

## Protocol Summary

_For a hgih-level overview of the Summa protocol, [see this](./README.md#overview-of-the-summa-proof-of-solvency-protocol)_.

### Overview of the MST-based Summa Version A (this report)

The core object in this version of the protocol is a Merkle sum tree (MST). Nodes in the tree all have two elements: a `hash`, and an array of `balances[b0, b1, .., bN]` where `N` is a global constant hardcoded to each instantiation of the protocol. In leaf nodes, the hash is  `H(user_id_transposed_into_a_field_element, [balances])` , while in middle nodes and the root, the hash is `H([balances], left_child_hash, right_child_hash)`. A balance b_i in a leaf node represents a user’s balance of the `i-th` currency, while in a non-leave node in the tree it represents the aggregated sum of the `i-th`  currency in all leaves that are descendants of said inner (or root) node. The ZK-friendly Poseidon hash function is used.

![mst in summa version a](./assets/mst.png?raw=true)

Merkle sum tree in [Summa Version A](https://summa.gitbook.io/summa/v/1/cryptographic-primitives/merkle-sum-tree)

User IDs are hashed in leaf nodes, and the root node (like all other inner nodes) contains only a hash an array of balances. These two elements are the public input to the verifier of the proof of grand sums, and as such no private user data are leaked. Merkle roots provided to users to feed as public input to the verifier of their inclusion proofs also do not leak any private data of other users for the same reason -blinding by hashing.

What is proven in-circuit by the entity are:

- The values of balances are valid and within the proper range, to combat potential attacks by overflowing the prime of the field
- The arithmetic of summation is valid
- The hash function is computed correctly

Each balance is range-checked in-circuit to a ceiling such that summing N users cannot possibly exceed the prime, with N being a safe maximum. The larger the ceiling the higher the prover cost (due to more decompositions during range-check). In summa, balances are range-checked to be ≤ 64 bits which are large enough for typical cryptocurrency balances while small enough to guarantee the summation of billions of users `N` cannot possibly overflow the much larger prime.

## Methodology
## Scope
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

### High

#### 1. High: [Possible Overflow in username in big_intify_username combined with calling big_uint_to_fp](https://github.com/zBlock-2/summa-solvency-diffie/issues/16) and [Guarantee usernames stays inside field](https://github.com/zBlock-2/summa-solvency-schneier/issues/13)
By: **sebastiantf, bbresearcher**, credit to **[Jin](https://github.com/sifnoc)** for guidance



A malicious prover could create usernames that overflow if two users have the same balance thus they can exclude one of the records from the data.


#### 2. High: [Sum balance overflow](https://github.com/zBlock-2/summa-solvency-diffie/issues/10)
By: **zeroqn**



There is no range check in the circuit for the sum of balances, which poses a risk of overflow.

Furthermore, since N_BYTES is not exposed in the contract, users must run `examples/gen_inclusion_verifier.rs` to obtain a warning message about the risk of overflow.

#### 3. High: [Inconsistency in range checks](https://github.com/zBlock-2/summa-solvency-Turing/issues/14)
By: **Y5Yash**

**Background**: The circuit checks for all levels in the tree if the sibling node's balance (and two leaf balances) is less than `m = 2 ** (NBYTES * 8)`. In the first look, this suggests that the max balance for the immediate parent of the leaf nodes at level 1 would be `2 * m` and the parent at level 2 would be `3 * m`, ..., and the max balance at the root will be `(NLEVEL - 1) * m`.

**Expected behavior**: Root's max balance is `(NLEVEL - 1) * m` as can be inferred from circuits/contracts.

### Low

#### 1. Low: [Mixed endian usage in code](https://github.com/zBlock-2/summa-solvency-diffie/issues/17)
By: **bbresearcher**

In the code here https://github.com/summa-dev/summa-solvency/blob/master/zk_prover/src/merkle_sum_tree/utils/operation_helpers.rs#L5-L17 ,
In specific big_intify_username uses big-endian and fp_to_big_uint uses little-endian.

### Informational

### 1. Informational: [Range check uses lookup_any instead of lookup](https://github.com/zBlock-2/summa-solvency-schneier/issues/18)
By: **obatirou**

The range check uses function `lookup_any` which was introduced in PSE fork to allow dynamic lookup by using a table expression. The table used for the range check do not change and is always the same: values from `0 to 2^8-1.` Hence a dynamic lookup is not necessary in this context. Usage of `lookup` should be preferred.

### 2. Informational: [`InclusionVerifier.yul`not generated](https://github.com/zBlock-2/summa-solvency-schneier/issues/16)
By: **flyingnobita**

It is mentioned in the Summa Book on the [summa-solvency page](https://summa.gitbook.io/summa-book/backend/summa-solvency) that `InclusionVerifier.sol` and `InclusionVerifier.yul` will be generated by the `gen_inclusion_verifier.rs` script. However, only `InclusionVerifier.sol` is generated.

> The script will generate a new `InclusionVerifier.sol` and `InclusionVerifier.yul` contracts in [`contracts/src`](https://github.com/summa-dev/summa-solvency/tree/master/contracts/src).

### 3. Informational: [Improvement to public inputs in contract](https://github.com/zBlock-2/summa-solvency-schneier/issues/12)
By: **sebastiantf**

The [`publicInputs`](https://github.com/zBlock-2/summa-solvency-schneier/blob/main/contracts/src/Summa.sol#L193) input to the contract is taken as an array. But its not a homogenous array. The expected [public inputs](https://summa.gitbook.io/summa-book/circuits/merkle-sum-tree-inclusion#public-inputs-outputs) are: user leaf hash, MST root followed by root balances:
https://github.com/zBlock-2/summa-solvency-schneier/blob/95d63fe1a55935542810138aa5d8de7f50f4e94b/contracts/src/Summa.sol#L188-L197

Since they are not homogenous or not values that have the same meaning, it might be better DX/UX to have them as separate meaningful inputs and combine them into an array within the function before submitting them to the verifier.

### 4. Informational: [Use only mapping for `addressOwnershipProofs`](https://github.com/zBlock-2/summa-solvency-schneier/issues/11)
By: **sebastiantf**

Currently both the array [`addressOwnershipProofs`](https://github.com/zBlock-2/summa-solvency-schneier/blob/95d63fe1a55935542810138aa5d8de7f50f4e94b/contracts/src/Summa.sol#L68) and the mapping [`_ownershipProofByAddress`](https://github.com/zBlock-2/summa-solvency-schneier/blob/95d63fe1a55935542810138aa5d8de7f50f4e94b/contracts/src/Summa.sol#L83C41-L83C65) are being used to track address ownership proofs. But the use of both seems unnecessary, inefficient and error-prone.

Didn't really notice any specific use-case being served by the array.

If there isn't really any use of the array, then I suppose we could remove the array, modify the mapping to store the `AddressOwnershipProof` struct and only use that. This is probably going to be a lot more efficient and simpler

### 5. Informational: [`Summa.sol` : Issue with `submitProofOfAddressOwnership()`](https://github.com/zBlock-2/summa-solvency-schneier/issues/7)
By: **zzzuhaibmohd**

**Issue #1**: `submitProofOfAddressOwnership()` does not allow to resubmit `AddressOwnershipProof` twice

Currently the require check at [Summa.sol#L120](https://github.com/zBlock-2/summa-solvency-schneier/blob/95d63fe1a55935542810138aa5d8de7f50f4e94b/contracts/src/Summa.sol#L120) allows the `onlyOwner` to add the `AddressOwnershipProof` associated with the `cexAddress` only once.

But imagine a scenario, wherein wrong signature or message was submitted during the first iteration, it that case, there is no way to update these values in the future.

**Issue #2**: `proofIndex` generation does no take into account the name of the chain

The root cause of the issue can be slightly linked to the first issue, The hash calculation at [Summa.sol#L117](https://github.com/zBlock-2/summa-solvency-schneier/blob/95d63fe1a55935542810138aa5d8de7f50f4e94b/contracts/src/Summa.sol#L117) does not take into consideration the chain as the input during calculation. While this might not look an issue for EVM chains. But when considering a multi chain architecture, using at least two input for generating the hash is a good practice. Currently thinking of a practical impact due to this.

### 6. Informational: [`Summa.sol` : Ownable: Does not implement 2-Step-Process for transferring ownership](https://github.com/zBlock-2/summa-solvency-schneier/issues/6)
By: **zzzuhaibmohd**

The contracts `Summa.sol` does not implement a 2-Step-Process for transferring ownership.
So ownership of the contract can easily be lost when making a mistake when transferring ownership.

While the probability if this happening is highly unlikely, it is better to follow best security measures.

### 7. Informational: [Potential `Summa::submitCommitment()` Gas limits](https://github.com/zBlock-2/summa-solvency-schneier/issues/4)
By: **sebastiantf**

[`Summa::submitCommitment()`](https://github.com/zBlock-2/summa-solvency-schneier/blob/main/contracts/src/Summa.sol#L144) takes in two arrays and [loops](https://github.com/zBlock-2/summa-solvency-schneier/blob/main/contracts/src/Summa.sol#L159) once over them:

https://github.com/zBlock-2/summa-solvency-schneier/blob/95d63fe1a55935542810138aa5d8de7f50f4e94b/contracts/src/Summa.sol#L146-L147

https://github.com/zBlock-2/summa-solvency-schneier/blob/95d63fe1a55935542810138aa5d8de7f50f4e94b/contracts/src/Summa.sol#L159-L171

`rootBalances` array contains the root balances of each cryptocurrency.
`cryptocurrencies` array contains details of each cryptocurrency: `name`, `chain`

There could be practical limitations to the number of rootBalances and cryptocurrencies that could be submitted in a single txn, imposed by block gas limits

According to [Coingecko](https://www.coingecko.com/en/exchanges/binance), Binance hosts 376 cryptocurrencies.

Current tests in the Summa contract codebase doesn't test this limitation as it only seems to be using 2 cryptocurrencies. See [here](https://github.com/zBlock-2/summa-solvency-schneier/blob/main/contracts/test/Summa.ts#L283), [here](https://github.com/zBlock-2/summa-solvency-schneier/blob/main/contracts/test/Summa.ts#L271) and [here](https://github.com/zBlock-2/summa-solvency-schneier/blob/main/zk_prover/examples/commitment_solidity_calldata.json#L4)

It shouldn't be too hard to add a test that tests such limits and it is WIP. But if it turns out that the contract is not able to handle such large number of cryptocurrencies, it would be a limitation to be aware of. It might be necessary to split the submission into multiple commitments for the same `timestamp`.

This would warrant if the circuit implementation supports this kind of split submission. Afaik, a single commitment is for the entire state of the exchange at a given time. If I'm wrong, please correct me, and feel free to close this issue.

### 8. Informational: [Magic numbers used in code of MST Circuit to create PoseidonChip](https://github.com/zBlock-2/summa-solvency-diffie/issues/15)
By: **bbresearcher**

There are hardcoded integers used in the construction of the Poseidon chips

### 9. Informational: [Review of the `Summa.sol` smart contract](https://github.com/zBlock-2/summa-solvency-diffie/issues/12)
By: **hrishibhat**

The Summa contract allows for centralized exchanges to prove to their users of the inclusion of their balances in the Merke Sum Tree.
The Summa contract is currently setup with the configuration that includes:
- mstLevels the number of levels to be used in an exchange's MST
- currenciesCount number of cryptocurrencies supported.
- balanceByteRange is the bytes used to represent the balance.

The CEX Owner submits proof of address ownership through `submitProofOfAddressOwnership`. This is an optimistic proof that these addresses belong to the exchange and the verification is done off-chain due to the large computational requirement that may not be feasible on-chain.

The CEX owner submits a commitment using `submitCommitment` about its liabilities in the form of a struct that uses the Merkle root, balances of the root, and the timestamp.

Once the commitment is submitted the user can then verify using the `verifyInclusionProof` function with relevant proof issued by the CEX and the public inputs against the exchange commitment for the respective timestamp that their balances are accurately represented in the CEX's Merkle tree. The verification is done by an external Inclusion verifier contract.

## Final remarks

## Recommendations
## Tool Configuration
## Appendix
### [Range check fuzzing](https://github.com/zBlock-2/summa-solvency-Turing/issues/15)
By: **teddav**
### [Installing and running halo2-analyzer](https://github.com/zBlock-2/summa-solvency-Turing/issues/12)
By: **teddav**
### Automated tests for dependency vulnerabilities and code quality [1](https://github.com/zBlock-2/summa-solvency-Turing/issues/11) [2](https://github.com/zBlock-2/summa-solvency-Turing/issues/10) [3](https://github.com/zBlock-2/summa-solvency-Turing/issues/9) [4](https://github.com/zBlock-2/summa-solvency-Turing/issues/8) [5](https://github.com/zBlock-2/summa-solvency-Turing/issues/7)
By: **sachindkagrawal15**
### [Set up the Fuzz testing suite in summa](https://github.com/zBlock-2/summa-solvency-Turing/issues/2)
By: **0xpanicError**
