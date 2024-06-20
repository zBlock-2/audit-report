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

- [yAcademy Summa Version A Review](#yacademy-summa-version-a-review)
  - [Table of Contents](#table-of-contents)
  - [Protocol Summary](#protocol-summary)
    - [Overview of the MST-based Summa Version A (this report)](#overview-of-the-mst-based-summa-version-a-this-report)
  - [Automated Testing](#automated-testing)
    - [Automated Analysis](#automated-analysis)
    - [Fuzz Testing](#fuzz-testing)
    - [Code Coverage](#code-coverage)
  - [Scope](#scope)
  - [Findings Explanation](#findings-explanation)
  - [Findings](#findings)
    - [High](#high)
    - [1. High: Missing username range check in `big_intify_username` \& `big_uint_to_fp`](#1-high-missing-username-range-check-in-big_intify_username--big_uint_to_fp)
      - [Recommendation](#recommendation)
      - [Refer](#refer)
    - [2. High: Sum balance overflow](#2-high-sum-balance-overflow)
      - [Refer](#refer-1)
    - [3. High: Inconsistency in range checks](#3-high-inconsistency-in-range-checks)
      - [Refer](#refer-2)
    - [Low](#low)
    - [1. Low: Mixed endian usage in code](#1-low-mixed-endian-usage-in-code)
      - [Refer](#refer-3)
    - [Informational](#informational)
    - [1. Informational: Range check uses lookup\_any instead of lookup](#1-informational-range-check-uses-lookup_any-instead-of-lookup)
      - [Refer](#refer-4)
    - [2. Informational: `InclusionVerifier.yul`not generated](#2-informational-inclusionverifieryulnot-generated)
      - [Refer](#refer-5)
    - [3. Informational: Improvement to public inputs in contract](#3-informational-improvement-to-public-inputs-in-contract)
    - [Refer](#refer-6)
    - [4. Informational: Use only mapping for `addressOwnershipProofs`](#4-informational-use-only-mapping-for-addressownershipproofs)
      - [Refer](#refer-7)
    - [5. Informational: `Summa.sol` : Issue with `submitProofOfAddressOwnership()`](#5-informational-summasol--issue-with-submitproofofaddressownership)
      - [Refer](#refer-8)
    - [6. Informational: `Summa.sol` : Ownable: Does not implement 2-Step-Process for transferring ownership](#6-informational-summasol--ownable-does-not-implement-2-step-process-for-transferring-ownership)
      - [Refer](#refer-9)
    - [7. Informational: Potential `Summa::submitCommitment()` Gas limits](#7-informational-potential-summasubmitcommitment-gas-limits)
      - [Refer](#refer-10)
    - [8. Informational: Magic numbers used in code of MST Circuit to create PoseidonChip](#8-informational-magic-numbers-used-in-code-of-mst-circuit-to-create-poseidonchip)
      - [Refer](#refer-11)
    - [9. Informational: Review of the `Summa.sol` smart contract](#9-informational-review-of-the-summasol-smart-contract)
      - [Refer](#refer-12)
  - [Final remarks](#final-remarks)
  - [Appendix](#appendix)
    - [A - Automated Analysis](#a---automated-analysis)
      - [1. Halo2-analyzer](#1-halo2-analyzer)
      - [2. Polyexen-demo](#2-polyexen-demo)
      - [3. NPM Audit](#3-npm-audit)
      - [4. Cargo Audit](#4-cargo-audit)
      - [5. Clippy](#5-clippy)
    - [B - Fuzz Testing](#b---fuzz-testing)
    - [C - Code Coverage](#c---code-coverage)
    - [Methodology](#methodology)

## Protocol Summary

_For a high-level overview of the Summa protocol, [see this](./README.md#overview-of-the-summa-proof-of-solvency-protocol)_.

### Overview of the MST-based Summa Version A (this report)

The core object in this version of the protocol is a Merkle sum tree (MST). Nodes in the tree all have two elements: a `hash`, and an array of `balances[b0, b1, .., bN]` where `N` is a global constant hardcoded to each instantiation of the protocol. In leaf nodes, the hash is `H(user_id_transposed_into_a_field_element, [balances])` , while in middle nodes and the root, the hash is `H([balances], left_child_hash, right_child_hash)`. A balance b_i in a leaf node represents a user’s balance of the `i-th` currency, while in a non-leave node in the tree it represents the aggregated sum of the `i-th` currency in all leaves that are descendants of said inner (or root) node. The ZK-friendly Poseidon hash function is used.

![mst in summa version a](./assets/mst.png?raw=true)

Figure 1: Merkle sum tree in [Summa Version A](https://summa.gitbook.io/summa/v/1/cryptographic-primitives/merkle-sum-tree)

User IDs are hashed in leaf nodes, and the root node (like all other inner nodes) contains only a hash an array of balances. The root hash and the aggregated `rootBalances` are the public inputs to the commitment from the CEX and as such no private user data are leaked. Merkle roots provided to users to feed as public input to the verifier of their inclusion proofs also do not leak any private data of other users for the same reason -blinding by hashing.

What is proven in-circuit by the entity are:

- The values of balances are valid and within the proper range, to combat potential attacks by overflowing the prime of the field
- The arithmetic of summation is valid
- The hash function is computed correctly

Each balance is range-checked in-circuit to a ceiling such that summing N users cannot possibly exceed the prime, with N being a safe maximum. The larger the ceiling the higher the prover cost (due to more decompositions during range-check). In summa, balances are range-checked to be ≤ 64 bits which are large enough for typical cryptocurrency balances while small enough to guarantee the summation of billions of users `N` cannot possibly overflow the much larger prime.

## Automated Testing

We use automated techniques to extensively test the security properties of software. We use both open-source static analysis and fuzzing utilities, along with tools developed in house, to perform automated testing of source code.

### Automated Analysis

We used the following tools in the automated testing phase of this project:

| Tool                                                           | Description                                                                                                                                                                                                                                                           | Section      |
| -------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ |
| [Halo2-analyzer](https://github.com/quantstamp/halo2-analyzer) | Halo2-analyzer / Korrekt employs a Satisfiability Modulo Theories (SMT) solver to try to find multiple satisfying models (i.e., private inputs) for a circuit and a (fixed) public input in order to determine if it is under-constrained.                            | [Appendix A.1](#1-halo2-analyzer) |
| [Polyexen-demo](https://github.com/ed255/polyexen-demo)        | Polyexen (Polynomial Expression Engine) transforms circuits designed with the Halo2 API into the Plonkish Arithmetization Format (Plaf). Plaf is designed to standardize plonkish circuits for reuse in components compatible with plonkish arithmetization.          | [Appendix A.2](#2-polyexen-demo) |
| [npm-audit](https://docs.npmjs.com/cli/v10/commands/npm-audit) | `npm audit` scans your project's dependencies for known security vulnerabilities, reports them with severity levels, and suggests fixes. It helps keep your Node.js application secure by identifying and addressing potential risks in your packages.                | [Appendix A.3](#3-npm-audit) |
| [cargo-audit](https://crates.io/crates/cargo-audit)            | `cargo audit` scans your Rust project's dependencies for known security vulnerabilities, reports them with severity levels, and suggests fixes. It helps keep your Rust application secure by identifying and addressing potential risks in your crates.              | [Appendix A.4](#4-cargo-audit) |
| [clippy](https://doc.rust-lang.org/clippy/)                    | `clippy` is a linter for Rust that checks your code for common mistakes and style issues. It provides helpful suggestions to improve your code quality and maintainability. Using clippy helps ensure your Rust code is clean, efficient, and follows best practices. | [Appendix A.5](#5-clippy) |

### Fuzz Testing

Fuzzing is a testing technique that tries to find bugs by repeatedly executing test cases and
mutating them. Classically, it is used in C/C++ codebases to detect segmentation faults,
buffer overflows, and other memory corruption vulnerabilities. In Rust, we can use it to find
runtime errors.

We set up a fuzz test suite using `cargo fuzz` for Merkle Sum Tree implementation, Range check & it’s utilities. [Appendix B](#b---fuzz-testing) contains a detailed description of the setup and deployment details.

### Code Coverage

We used [cargo-llvm-cov](https://github.com/taiki-e/cargo-llvm-cov) to generate LLVM source-based code coverage. Refer to [Appendix C](#c---code-coverage) for more detailed information on testing & code coverage.

## Scope

The security review was limited to commit [95d63fe](https://github.com/summa-dev/summa-solvency/tree/95d63fe1a55935542810138aa5d8de7f50f4e94b). The scope of the review consisted of the following files and folders at the specific commit:

- `/zk_prover/*`
- `/contracts/src/Summa.sol`
- `/backend/*`

After the findings were presented to the Summa team, fixes were made and included in several PRs.

This code review is for identifying potential vulnerabilities in the code. The reviewers did not investigate security practices or operational security and assumed that privileged parties could be trusted. The reviewers did not evaluate the security of the code relative to a standard or specification. The review may not have identified all potential attack vectors or areas of vulnerability.

yAcademy and the auditors make no warranties regarding the security of the code and do not warrant that the code is free from defects. yAcademy and the auditors do not represent nor imply to third parties that the code has been audited nor that the code is free from defects. By deploying or using the code, Summa Solvency and users of the contracts/circuits agree to use the code at their own risk.

## Findings Explanation

Findings are broken down into sections by their respective Impact:

- Critical, High, Medium, Low Impact
  - These are findings that range from attacks that may cause loss of funds, a break in the soundness, zero-knowledge, completeness of the system, proof malleability, or cause any unintended consequences/actions that are outside the scope of the requirements
- Informational
  - Findings including Recommendations and best practices

---

## Findings

### High

### 1. High: Missing username range check in `big_intify_username` & `big_uint_to_fp`

```rust
/// Return a BigUint representation of the username
pub fn big_intify_username(username: &str) -> BigUint {
    let utf8_bytes = username.as_bytes();
    BigUint::from_bytes_be(utf8_bytes)
}
/// Converts a BigUint to a Field Element
pub fn big_uint_to_fp(big_uint: &BigUint) -> Fp {
    Fp::from_str_vartime(&big_uint.to_str_radix(10)[..]).unwrap()
}
```
A malicious prover could create usernames that overflow if two users have the same balance thus they can exclude one of the records from the data.

#### Recommendation

Include a range check inside the circuit or inside the smart contract. The range checks are to ensure that all usernames were less than the snark scalar field order so two users don't end up with the same identity in the Merkle sum tree

#### Refer
- [Guarantee usernames stays inside field](https://github.com/zBlock-2/summa-solvency-schneier/issues/13) by [sebastiantf]()
- [Possible Overflow in username in big_intify_username combined with calling big_uint_to_fp](https://github.com/zBlock-2/summa-solvency-diffie/issues/16) by [parsley]()

### 2. High: Sum balance overflow

There is no in-circuit range check for the sum of balances, which poses a risk of overflow. Since `N_BYTES` is not exposed in the contract, users must run `examples/gen_inclusion_verifier.rs` to obtain a warning message about the risk of overflow.

#### Refer
- [Sum Balance Overflow](https://github.com/zBlock-2/summa-solvency-diffie/issues/10) by [zeroqn]()

### 3. High: Inconsistency in range checks

The circuit checks for all levels in the tree if the sibling node's balance (and two leaf balances) is less than `m = 2 ** (NBYTES * 8)`. In the first look, this suggests that the max balance for the immediate parent of the leaf nodes at level 1 would be `2 * m` and the parent at level 2 would be `3 * m`, ..., and the max balance at the root will be `(NLEVEL - 1) * m`.

Root's max balance is `(NLEVEL - 1) * m` as can be inferred from circuits/contracts.

#### Refer

- [Inconsistency in range checks](https://github.com/zBlock-2/summa-solvency-Turing/issues/14) by [y5yash]()

### Low

### 1. Low: Mixed endian usage in code

In the code [here](https://github.com/summa-dev/summa-solvency/blob/master/zk_prover/src/merkle_sum_tree/utils/operation_helpers.rs#L5-L17), `big_intify_username` uses big-endian and `fp_to_big_uint` uses little-endian.

#### Refer

- [Mixed endian usage in code](https://github.com/zBlock-2/summa-solvency-diffie/issues/17) by [bbresearcher]()

### Informational

### 1. Informational: Range check uses lookup_any instead of lookup

The range check uses function `lookup_any` which was introduced in PSE fork to allow dynamic lookup by using a table expression. The table used for the range check do not change and is always the same: values from `0 to 2^8-1.` Hence a dynamic lookup is not necessary in this context. Usage of `lookup` should be preferred.

#### Refer

- [Range check uses lookup_any instead of lookup](https://github.com/zBlock-2/summa-solvency-schneier/issues/18) By  [obatirou]()

### 2. Informational: `InclusionVerifier.yul`not generated

It is mentioned in the Summa Book on the [summa-solvency page](https://summa.gitbook.io/summa-book/backend/summa-solvency) that `InclusionVerifier.sol` and `InclusionVerifier.yul` will be generated by the `gen_inclusion_verifier.rs` script. However, only `InclusionVerifier.sol` is generated.

> The script will generate a new `InclusionVerifier.sol` and `InclusionVerifier.yul` contracts in [`contracts/src`](https://github.com/summa-dev/summa-solvency/tree/master/contracts/src).

#### Refer 

- [`InclusionVerifier.yul`not generated](https://github.com/zBlock-2/summa-solvency-schneier/issues/16) by [flyingnobita]()

### 3. Informational: Improvement to public inputs in contract

The [`publicInputs`](https://github.com/zBlock-2/summa-solvency-schneier/blob/main/contracts/src/Summa.sol#L193) input to the contract is taken as an array. But its not a homogenous array. The expected [public inputs](https://summa.gitbook.io/summa-book/circuits/merkle-sum-tree-inclusion#public-inputs-outputs) are: user leaf hash, MST root followed by root balances:
https://github.com/zBlock-2/summa-solvency-schneier/blob/95d63fe1a55935542810138aa5d8de7f50f4e94b/contracts/src/Summa.sol#L188-L197

Since they are not homogenous or not values that have the same meaning, it might be better DX/UX to have them as separate meaningful inputs and combine them into an array within the function before submitting them to the verifier.

### Refer 

- [Improvement to public inputs in contract](https://github.com/zBlock-2/summa-solvency-schneier/issues/12) By [sebastiantf]()

### 4. Informational: Use only mapping for `addressOwnershipProofs`

Currently both the array [`addressOwnershipProofs`](https://github.com/zBlock-2/summa-solvency-schneier/blob/95d63fe1a55935542810138aa5d8de7f50f4e94b/contracts/src/Summa.sol#L68) and the mapping [`_ownershipProofByAddress`](https://github.com/zBlock-2/summa-solvency-schneier/blob/95d63fe1a55935542810138aa5d8de7f50f4e94b/contracts/src/Summa.sol#L83C41-L83C65) are being used to track address ownership proofs. But the use of both seems unnecessary, inefficient and error-prone. Didn't really notice any specific use-case being served by the array.

If there isn't really any use of the array, then I suppose we could remove the array, modify the mapping to store the `AddressOwnershipProof` struct and only use that. This is probably going to be a lot more efficient and simpler.

#### Refer 

- [Use only mapping for `addressOwnershipProofs`](https://github.com/zBlock-2/summa-solvency-schneier/issues/11) by [sebastiantf]()

### 5. Informational: `Summa.sol` : Issue with `submitProofOfAddressOwnership()`

**Issue #1**: `submitProofOfAddressOwnership()` does not allow to resubmit `AddressOwnershipProof` twice

Currently the require check at [Summa.sol#L120](https://github.com/zBlock-2/summa-solvency-schneier/blob/95d63fe1a55935542810138aa5d8de7f50f4e94b/contracts/src/Summa.sol#L120) allows the `onlyOwner` to add the `AddressOwnershipProof` associated with the `cexAddress` only once.

But imagine a scenario, wherein wrong signature or message was submitted during the first iteration, it that case, there is no way to update these values in the future.

**Issue #2**: `proofIndex` generation does no take into account the name of the chain

The root cause of the issue can be slightly linked to the first issue, The hash calculation at [Summa.sol#L117](https://github.com/zBlock-2/summa-solvency-schneier/blob/95d63fe1a55935542810138aa5d8de7f50f4e94b/contracts/src/Summa.sol#L117) does not take into consideration the chain as the input during calculation. While this might not look an issue for EVM chains. But when considering a multi chain architecture, using at least two input for generating the hash is a good practice. Currently thinking of a practical impact due to this.

#### Refer

- [`Summa.sol` : Issue with `submitProofOfAddressOwnership()`](https://github.com/zBlock-2/summa-solvency-schneier/issues/7) by [zzzuhaibmohd]()

### 6. Informational: `Summa.sol` : Ownable: Does not implement 2-Step-Process for transferring ownership

The contracts `Summa.sol` does not implement a 2-Step-Process for transferring ownership.
So ownership of the contract can easily be lost when making a mistake when transferring ownership.

While the probability if this happening is highly unlikely, it is better to follow best security measures.

#### Refer

- [`Summa.sol` : Ownable: Does not implement 2-Step-Process for transferring ownership](https://github.com/zBlock-2/summa-solvency-schneier/issues/6) by [zzzuhaibmohd]()

### 7. Informational: Potential `Summa::submitCommitment()` Gas limits

[`Summa::submitCommitment()`](https://github.com/zBlock-2/summa-solvency-schneier/blob/main/contracts/src/Summa.sol#L144) takes in two arrays and [loops](https://github.com/zBlock-2/summa-solvency-schneier/blob/main/contracts/src/Summa.sol#L159) once over them:

https://github.com/zBlock-2/summa-solvency-schneier/blob/95d63fe1a55935542810138aa5d8de7f50f4e94b/contracts/src/Summa.sol#L146-L147

https://github.com/zBlock-2/summa-solvency-schneier/blob/95d63fe1a55935542810138aa5d8de7f50f4e94b/contracts/src/Summa.sol#L159-L171

`rootBalances` array contains the root balances of each cryptocurrency.
`cryptocurrencies` array contains details of each cryptocurrency: `name`, `chain`

There could be practical limitations to the number of rootBalances and cryptocurrencies that could be submitted in a single txn, imposed by block gas limits

According to [Coingecko](https://www.coingecko.com/en/exchanges/binance), Binance hosts 376 cryptocurrencies.

Current tests in the Summa contract codebase doesn't test this limitation as it only seems to be using 2 cryptocurrencies. See [here](https://github.com/zBlock-2/summa-solvency-schneier/blob/main/contracts/test/Summa.ts#L283), [here](https://github.com/zBlock-2/summa-solvency-schneier/blob/main/contracts/test/Summa.ts#L271) and [here](https://github.com/zBlock-2/summa-solvency-schneier/blob/main/zk_prover/examples/commitment_solidity_calldata.json#L4)

It shouldn't be too hard to add a test that tests such limits and it is WIP. But if it turns out that the contract is not able to handle such large number of cryptocurrencies, it would be a limitation to be aware of. It might be necessary to split the submission into multiple commitments for the same `timestamp`.

This would warrant if the circuit implementation supports this kind of split submission. Afaik, a single commitment is for the entire state of the exchange at a given time. 

#### Refer

- [Potential `Summa::submitCommitment()` Gas limits](https://github.com/zBlock-2/summa-solvency-schneier/issues/4) by [sebastiantf]()

### 8. Informational: Magic numbers used in code of MST Circuit to create PoseidonChip

There are hardcoded integers used in the construction of the Poseidon chips

#### Refer 

- [Magic numbers used in code of MST Circuit to create PoseidonChip](https://github.com/zBlock-2/summa-solvency-diffie/issues/15) by [bbresearcher]()

### 9. Informational: Review of the `Summa.sol` smart contract

The Summa contract allows for centralized exchanges to prove to their users of the inclusion of their balances in the Merke Sum Tree.
The Summa contract is currently setup with the configuration that includes:

- mstLevels the number of levels to be used in an exchange's MST
- currenciesCount number of cryptocurrencies supported.
- balanceByteRange is the bytes used to represent the balance.

The CEX Owner submits proof of address ownership through `submitProofOfAddressOwnership`. This is an optimistic proof that these addresses belong to the exchange and the verification is done off-chain due to the large computational requirement that may not be feasible on-chain.

The CEX owner submits a commitment using `submitCommitment` about its liabilities in the form of a struct that uses the Merkle root, balances of the root, and the timestamp.

Once the commitment is submitted the user can then verify using the `verifyInclusionProof` function with relevant proof issued by the CEX and the public inputs against the exchange commitment for the respective timestamp that their balances are accurately represented in the CEX's Merkle tree. The verification is done by an external Inclusion verifier contract.

#### Refer

- [Review of the `Summa.sol` smart contract](https://github.com/zBlock-2/summa-solvency-diffie/issues/12) by [hrishibhat]()

## Final remarks

## Appendix

### A - Automated Analysis

#### 1. Halo2-analyzer

Halo2-analyzer / Korrekt employs a Satisfiability Modulo Theories (SMT) solver to try to find multiple satisfying models (i.e., private inputs) for a circuit and a (fixed) public input in order to determine if it is under-constrained. It works by collecting all the gates and regions using a custom implementation (`AnalyticLayouter`) of the `Layouter` trait. For every `region`, we have a set of enabled selectors and assigned cells (but with dummy values), likewise for every `gate`, we have a set of polynomials for each gate. This simple tool performs three checks :

- **Unused Gate -** Check that for every gate there exists a region in which it is not identically zero.
- **Unused Column -** Check that every column occurs in some polynomial.
- **Unconstrained Cell -** Check that for every assigned cell in the region, it occurs in a polynomial which is not identically zero over this region. This means that part of the witness is not constrained -- this is almost certainly a bug.

We used halo2-analyzer to search for unused gates, underconstrained cells and unused columns in the circuits. Here are the results :

1. Unused Gate

```bash
Finished analysis: 2 unused gates found.
unused gate: "partial rounds" (consider removing the gate or checking selectors in regions)
unused gate: "partial rounds" (consider removing the gate or checking selectors in regions)
```

The gate [`partial rounds`](https://github.com/summa-dev/halo2/blob/main/halo2_gadgets/src/poseidon/pow5.rs#L117) is defined in the `Pow5Chip` of the Poseidon circuit. Since, the gate isn’t used, it is recommended that the gate be removed.

2. Unused Columns

```bash
Finished analysis: 0 unused columns found.
```

3. Underconstrained Cell

```bash
Finished analysis: 3566 unconstrained cells found.
unconstrained cell in "assign entry username" region: Column { index: 0, column_type: Advice } (rotation: 0) -- very likely a bug.
unconstrained cell in "assign entry balance" region: Column { index: 1, column_type: Advice } (rotation: 0) -- very likely a bug.
unconstrained cell in "assign entry balance" region: Column { index: 1, column_type: Advice } (rotation: 0) -- very likely a bug.
unconstrained cell in "initial state for domain ConstantLength<3>" region: Column { index: 1, column_type: Advice } (rotation: 0) -- very likely a bug.
unconstrained cell in "initial state for domain ConstantLength<3>" region: Column { index: 0, column_type: Advice } (rotation: 0) -- very likely a bug.
unconstrained cell in "permute state" region: Column { index: 0, column_type: Advice } (rotation: 0) -- very likely a bug.
...
unconstrained cell in "assign sibling leaf node username" region: Column { index: 0, column_type: Advice } (rotation: 0) -- very likely a bug.
unconstrained cell in "assign sibling leaf balance" region: Column { index: 1, column_type: Advice } (rotation: 0) -- very likely a bug.
unconstrained cell in "initial state for domain ConstantLength<3>" region: Column { index: 1, column_type: Advice } (rotation: 0) -- very likely a bug.
unconstrained cell in "initial state for domain ConstantLength<3>" region: Column { index: 0, column_type: Advice } (rotation: 0) -- very likely a bug.
unconstrained cell in "permute state" region: Column { index: 1, column_type: Advice } (rotation: 0) -- very likely a bug.
...
unconstrained cell in "sum nodes balances per currency" region: Column { index: 0, column_type: Advice } (rotation: 0) -- very likely a bug.
unconstrained cell in "initial state for domain ConstantLength<4>" region: Column { index: 0, column_type: Advice } (rotation: 0) -- very likely a bug.
unconstrained cell in "initial state for domain ConstantLength<4>" region: Column { index: 1, column_type: Advice } (rotation: 0) -- very likely a bug.
unconstrained cell in "permute state" region: Column { index: 1, column_type: Advice } (rotation: 0) -- very likely a bug.
...
unconstrained cell in "assign value to perform range check" region: Column { index: 0, column_type: Advice } (rotation: 0) -- very likely a bug.
unconstrained cell in "sum nodes balances per currency" region: Column { index: 0, column_type: Advice } (rotation: 0) -- very likely a bug.
unconstrained cell in "initial state for domain ConstantLength<4>" region: Column { index: 1, column_type: Advice } (rotation: 0) -- very likely a bug.
unconstrained cell in "initial state for domain ConstantLength<4>" region: Column { index: 0, column_type: Advice } (rotation: 0) -- very likely a bug.
unconstrained cell in "permute state" region: Column { index: 0, column_type: Advice } (rotation: 0) -- very likely a bug.
...
unconstrained cell in "assign sibling node balance" region: Column { index: 1, column_type: Advice } (rotation: 0) -- very likely a bug.
unconstrained cell in "assign sibling node balance" region: Column { index: 1, column_type: Advice } (rotation: 0) -- very likely a bug.
unconstrained cell in "assign sibling left hash" region: Column { index: 2, column_type: Advice } (rotation: 0) -- very likely a bug.
unconstrained cell in "assign sibling right hash" region: Column { index: 2, column_type: Advice } (rotation: 0) -- very likely a bug.
unconstrained cell in "initial state for domain ConstantLength<4>" region: Column { index: 1, column_type: Advice } (rotation: 0) -- very likely a bug.
unconstrained cell in "initial state for domain ConstantLength<4>" region: Column { index: 0, column_type: Advice } (rotation: 0) -- very likely a bug.
unconstrained cell in "permute state" region: Column { index: 2, column_type: Advice } (rotation: 0) -- very likely a bug.
unconstrained cell in "permute state" region: Column { index: 1, column_type: Advice } (rotation: 0) -- very likely a bug.
unconstrained cell in "permute state" region: Column { index: 1, column_type: Advice } (rotation: 0) -- very likely a bug.
```

Out of the `3566` underconstrained cells found, these are the common weaknesses pointed out :

- unconstrained cell in `"assign entry username"`, `"assign entry balance"`, `"assign sibling leaf node username"`, `"assign sibling leaf balance"`, `"assign sibling node balance"`, `"assign sibling left hash"`, `"assign sibling right hash"` regions. The `assign_value_to_witness` method from the `CircuitBase` trait is used to assign the values during synthesis. A malicious prover can tweak these values & construct a completely new subset of a Merkle tree. This is a known issue.

Here’s the complete [report](https://github.com/zBlock-2/summa-report/blob/main/underconstrained_cells.md).

#### 2. Polyexen-demo

Polyexen (Polynomial Expression Engine) transforms circuits designed with the Halo2 API into the Plonkish Arithmetization Format (Plaf). Plaf is designed to standardize plonkish circuits for reuse in components compatible with plonkish arithmetization. The current implementation supports serializing a Plaf circuit into two parts:

- A CSV file containing the fixed column values.
- A TOML file encompassing the rest of the circuit definition, including:
  - Lookup constraints
  - Gate constraints
  - Copy constraints

We used polyexen-demo to debug the assignments & double check the constraints. Here’s the output :

- Fixed Columns - [CSV](https://github.com/zBlock-2/summa-report/blob/main/mst_fixed.csv)
- Lookup constraints - [mst_lookups.toml](https://github.com/zBlock-2/summa-report/blob/main/mst_lookups.toml)
- Gate constraints - [mst_polys.toml](https://github.com/zBlock-2/summa-report/blob/main/mst_polys.toml)
- Copy constraints - [mst.toml](https://github.com/zBlock-2/summa-report/blob/main/mst.toml)

#### 3. NPM Audit

`npm audit` scans your project's dependencies for known security vulnerabilities, reports them with severity levels, and suggests fixes. It helps keep your Node.js application secure by identifying and addressing potential risks in your packages. View the complete report of security vulnerabilities in the `contracts` package [here](https://github.com/zBlock-2/audit-report/blob/main/appendix/V1/npm-audit/output.md)

#### 4. Cargo Audit

`cargo audit` scans your Rust project's dependencies for known security vulnerabilities, reports them with severity levels, and suggests fixes. It helps keep your Rust application secure by identifying and addressing potential risks in your crates. View the complete report of security vulnerabilities in `zk-prover` and `backend` [here](https://github.com/zBlock-2/audit-report/blob/main/appendix/V1/cargo-audit/output.md).

#### 5. Clippy

`clippy` is a linter for Rust that checks your code for common mistakes and style issues. It provides helpful suggestions to improve your code quality and maintainability. Using `clippy` helps ensure your Rust code is clean, efficient, and follows best practices. Here's the [report](https://github.com/zBlock-2/audit-report/blob/main/appendix/V1/clippy/output.md).

### B - Fuzz Testing

Fuzzing is a testing technique that tries to find bugs by repeatedly executing test cases and mutating them. Classically, it is used in C/C++ codebases to detect segmentation faults, buffer overflows, and other memory corruption vulnerabilities. In Rust, we can use it to find runtime errors.

We set up a fuzz test suite using `cargo fuzz` (which uses `libfuzzer-sys`) for Merkle Sum Tree implementation, Range check & it’s utilities. We initialized a basic fuzz suite in [PR#18](https://github.com/zBlock-2/summa-solvency-diffie/pull/18) and in [PR#3](https://github.com/zBlock-2/summa-solvency-Turing/pull/3) with a [Setup Tutorial](https://github.com/zBlock-2/summa-solvency-Turing/pull/3/files#diff-f91a5e0dcdf679d481a8eaf944ca24572fe682b1ca746dd39136b612c8dcaa55). Furthermore, fuzz tests for `merkle_sum_tree`, `utils`, `csv_parser` were included in [PR#6](https://github.com/zBlock-2/summa-solvency-diffie/pull/6).

Upon fuzzing the `utils` in [PR#6](https://github.com/zBlock-2/summa-solvency-diffie/pull/6/files#diff-9412e8e264746a8c097d218a1de08b4db7b899a6479e16486fced43f653cfee2), there seems to be a little discrepency in the bytes to Fp conversion cycle. An assertion fails when comparing an empty string's bytes ([]) to the result of converting that empty string to a `BigUint` and then back to bytes, which results in [0]. This happens because converting an empty string to a `BigUint` using `BigUint::from(0u8)` creates a `BigUint` representing 0, which converts back to [0] instead of an empty array. To fix this, we need to ensure that converting an empty string to a `BigUint` and back maintains consistency with the original input.

### C - Code Coverage

![alt text](./assets/image.png)

We raised the following pull requests to increase code coverage & emphasize on testing.

- [PR#3](https://github.com/zBlock-2/summa-solvency-diffie/pull/3) to increase code coverage for `merkle_sum_tree`
- [PR#17](https://github.com/zBlock-2/summa-solvency-schneier/pull/17) to add end-to-end testing with full prover and verifier (instead of mock prover).
- [PR#8](https://github.com/zBlock-2/summa-solvency-schneier/pull/8/files) to include cost estimation for circuits using `CircuitCost`
- [PR#5](https://github.com/zBlock-2/summa-solvency-schneier/pull/5) is a stress test to determine the potential gas limits of `Summa::submitCommitment()`

### [Methodology](#methodology)

The audit employed a blend of automated tools and manual examination conducted by the fellows and residents. Techniques included detailed code reviews, static and dynamic analysis, fuzzing, and penetration testing to ensure a thorough validation of the protocol’s security measures.

- **Tool Integration:**
The audit utilized several specialized tools, each tailored to assess different aspects of the protocol:
    - **Halo2-analyzer**: Verified all circuit constraints.
    - **Polyexen-demo**: Standardized circuit formats for clarity and reusability.
    - **Highlighter**: Identified potential code issues needing closer examination.
    - **NPM and Cargo Audits**: Checked dependencies for known vulnerabilities.
    - **Clippy**: Ensured Rust code quality and best practices.
- **Analytical Techniques:**
The audit encompassed both static and dynamic analyses to provide a comprehensive security assessment:
    - **Static Analysis**: Examined the source code for vulnerabilities without execution.
    - **Dynamic Analysis**: Tested the protocol in operation to identify runtime issues.
- **Expert Review:**
We conducted in-depth manual reviews to evaluate complex components and integrations, providing a crucial layer of scrutiny beyond automated tools.
- **Feedback and Improvements:**
An iterative feedback loop with the Summa’s development team allowed for the immediate addressing and re-evaluation of any issues found, ensuring all fixes were effectively implemented.
- **Documentation:**
Each phase of the audit was thoroughly documented, with detailed reports on tool outputs, expert insights, and overall findings, culminating in a comprehensive final report that outlined vulnerabilities, impacts, and recommended actions.