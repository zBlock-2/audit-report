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

- [yAcademy Summa Version B Review](#yacademy-summa-version-b-review)
  - [Table of Contents](#table-of-contents)
  - [Protocol Summary](#protocol-summary)
    - [Overview of the KZG-based Implementation of the Summa Proof of Solvency Protocol](#overview-of-the-kzg-based-implementation-of-the-summa-proof-of-solvency-protocol)
  - [Scope](#scope)
  - [Automated testing](#automated-testing)
    - [Automated Analysis](#automated-analysis)
  - [Findings Explanation](#findings-explanation)
  - [Findings](#findings)
    - [High](#high)
    - [1. High: The inclusion proof reveals whether a user holds a specific token](#1-high-the-inclusion-proof-reveals-whether-a-user-holds-a-specific-token)
    - [2. High: Wrong verifying key contract permutation length can be considered valid by `validateVKPermutationsLength`](#2-high-wrong-verifying-key-contract-permutation-length-can-be-considered-valid-by-validatevkpermutationslength)
    - [3. High: Fake user Inclusion Proof verified in contract](#3-high-fake-user-inclusion-proof-verified-in-contract)
    - [4. High: Username overflow](#4-high-username-overflow)
    - [Medium](#medium)
    - [1. Medium: The return value of `GrandSumVerifier` should be tested](#1-medium-the-return-value-of-grandsumverifier-should-be-tested)
    - [Low](#low)
    - [1. Low: Incorrect permutation length in `validateVKPermuationsLength`](#1-low-incorrect-permutation-length-in-validatevkpermuationslength)
    - [2. Low: CSV parsing allows duplicate crypto tokens](#2-low-csv-parsing-allows-duplicate-crypto-tokens)
    - [3. Low: The return value of `GrandSumVerifier` should be tested](#3-low-the-return-value-of-grandsumverifier-should-be-tested)
    - [4. Low: Range check tests are unreliable regarding the number of overflows](#4-low-range-check-tests-are-unreliable-regarding-the-number-of-overflows)
    - [Informational](#informational)
    - [1. Informational: Security concerns in `Summa.sol`](#1-informational-security-concerns-in-summasol)
    - [2. Informational: Update memory locations in verifiers to save gas](#2-informational-update-memory-locations-in-verifiers-to-save-gas)
    - [3. Informational: Dynamic Errors not handled in Box of Errors for Function Results](#3-informational-dynamic-errors-not-handled-in-box-of-errors-for-function-results)
    - [4. Informational: Use of unwrap in core files and improper error handling](#4-informational-use-of-unwrap-in-core-files-and-improper-error-handling)
  - [Final remarks](#final-remarks)
  - [Appendix](#appendix)
    - [A - Automated Analysis](#a---automated-analysis)
      - [1. Halo2-analyzer](#1-halo2-analyzer)
    - [1. Unused Gate](#1-unused-gate)
    - [2. Unused columns](#2-unused-columns)
    - [3. Underconstrained Cells](#3-underconstrained-cells)
      - [2. Polyexen-demo](#2-polyexen-demo)
      - [3. Highlighter](#3-highlighter)
      - [4. NPM Audit](#4-npm-audit)
      - [5. Cargo Audit](#5-cargo-audit)
      - [6. Clippy](#6-clippy)
    - [Methodology](#methodology)

## Protocol Summary

_For a hgih-level overview of the Summa protocol, [see this](./README.md#overview-of-the-summa-proof-of-solvency-protocol)_.

### Overview of the KZG-based Implementation of the Summa Proof of Solvency Protocol

The 2nd iteration of the Summa protocol aims to lower proving and verification costs while preserving the same privacy guarantees. The core primitive in this iteration of the protocol is a [cleverly](https://drive.google.com/drive/folders/1081DlyvigZykFaWmI3JCSknpnUEw0z67) [interpolated](https://drive.google.com/drive/folders/1k-8HDie_adueyjY0-TIhHaLawtT6p27d) polynomial that brings about performance efficiencies compared to the hashing-intensive MST in Version A. The overall flow of the protocol remains the same: the prover produces a proof of grand sum, and individual proofs of inclusion for each user. The reliance on large number of users verifying their inclusion proofs also remains, and the same privacy guarantees are also preserved.

A Lagrange basis polynomial is interpolated such that at `x=0` it evaluates to the grand sum of all liabilities (more precisely, the constant coefficient of the polynomial multiplied by the number of users = grand sum), while at `(w_j)^i` , where `w_j` is the `j-th` root of unity and `i` is a user-unique index, _it evaluates to the balance of the `i-th` user_. The construction takes advantage of the fact that the sum of roots of unity is zero in order to “encode” the desired evaluations into the polynomial.

The prover produces a commitment `C` to the polynomial and:

- An opening [proof](https://github.com/summa-dev/summa-solvency/blob/bd3a3d4d8fda6661e4eede00cf6176d66cb00858/prover/src/circuits/tests.rs#L167) at `x=0` to [the public](https://github.com/summa-dev/summa-solvency/blob/fec83a747ead213261aecfaf4a01b43fff9731ee/contracts/src/Summa.sol#L230)
- An opening proof at `(w_j)^i` for the `i-th` user, who can can verify inclusion not only that the evaluation is correct but also that the commitment `C` they evaluated against is the same as the public commitment.

## Scope

The security review was limited to commit [fec83a747](https://github.com/summa-dev/summa-solvency/tree/95d63fe1a55935542810138aa5d8de7f50f4e94b). The scope of the review consisted of the following files and folders at the specific commit:

- `/backend/*`
- `/contracts/*`
- `/csv/*`
- `/prover/*` excluding [amortized KZG opening approach](https://github.com/summa-dev/summa-solvency/blob/fec83a747ead213261aecfaf4a01b43fff9731ee/prover/src/circuits/tests.rs#L31) and  related code in [utils](https://github.com/summa-dev/summa-solvency/blob/fec83a747ead213261aecfaf4a01b43fff9731ee/prover/src/circuits/utils.rs#L236-L307)

After the findings were presented to the Summa team, fixes were made and included in several PRs.

This code review is for identifying potential vulnerabilities in the code. The reviewers did not investigate security practices or operational security and assumed that privileged parties could be trusted. The reviewers did not evaluate the security of the code relative to a standard or specification. The review may not have identified all potential attack vectors or areas of vulnerability.

yAcademy and the auditors make no warranties regarding the security of the code and do not warrant that the code is free from defects. yAcademy and the auditors do not represent nor imply to third parties that the code has been audited nor that the code is free from defects. By deploying or using the code, Summa Solvency and users of the contracts/circuits agree to use the code at their own risk.

## Automated testing

We use automated techniques to extensively test the security properties of software. We use both open-source static analysis and fuzzing utilities, along with tools developed in house, to perform automated testing of source code.

### Automated Analysis

We used the following tools in the automated testing phase of this project:

| Tool                                                                          | Description                                                                                                                                                                                                                                                           | Section                           |
| ----------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------- |
| [Halo2-analyzer](https://github.com/quantstamp/halo2-analyzer)                | Halo2-analyzer / Korrekt employs a Satisfiability Modulo Theories (SMT) solver to try to find multiple satisfying models (i.e., private inputs) for a circuit and a (fixed) public input in order to determine if it is under-constrained.                            | [Appendix A.1](#1-halo2-analyzer) |
| [Polyexen-demo](https://github.com/ed255/polyexen-demo)                       | Polyexen (Polynomial Expression Engine) transforms circuits designed with the Halo2 API into the Plonkish Arithmetization Format (Plaf). Plaf is designed to standardize plonkish circuits for reuse in components compatible with plonkish arithmetization.          | [Appendix A.2](#2-polyexen-demo)  |
| [Highlighter](https://github.com/bbresearcher/ZK/tree/main/Halo2/Highlighter) | Highlighter is a python script which runs against a HALO2 (or Rust) project and will highlight any code that may need to be checked a bit closer                                                                                                                      | [Appendix A.3](#3-highlighter)    |
| [npm-audit](https://docs.npmjs.com/cli/v10/commands/npm-audit)                | `npm audit` scans your project's dependencies for known security vulnerabilities, reports them with severity levels, and suggests fixes. It helps keep your Node.js application secure by identifying and addressing potential risks in your packages.                | [Appendix A.3](#4-npm-audit)      |
| [cargo-audit](https://crates.io/crates/cargo-audit)                           | `cargo audit` scans your Rust project's dependencies for known security vulnerabilities, reports them with severity levels, and suggests fixes. It helps keep your Rust application secure by identifying and addressing potential risks in your crates.              | [Appendix A.4](#5-cargo-audit)    |
| [clippy](https://doc.rust-lang.org/clippy/)                                   | `clippy` is a linter for Rust that checks your code for common mistakes and style issues. It provides helpful suggestions to improve your code quality and maintainability. Using clippy helps ensure your Rust code is clean, efficient, and follows best practices. | [Appendix A.5](#5-cargo-audit)    |

## Findings Explanation

Findings are broken down into sections by their respective Impact:

- Critical, High, Medium, Low Impact
  - These are findings that range from attacks that may cause loss of funds, a break in the soundness, zero-knowledge, completeness of the system, proof malleability, or cause any unintended consequences/actions that are outside the scope of the requirements
- Informational
  - Findings including Recommendations and best practices

---

## Findings

### High

### 1. High: [The inclusion proof reveals whether a user holds a specific token](https://github.com/zBlock-2/summa-solvency/issues/21)

By: **qpzm**

A sniffer can see whether a user has a specific token because balance polynomials are unblinded.
A sniffer, for example, may be an web3 api endpoint who can see a user's inputs for `verifyInclusionProof`.

**Process**:

1. Assume $B(w^i) = 0$ and check $e([B(s)]_1 - [0]_1, [1]_2) == e([Q_B(w^i)]_1, [s]_2 - [w^i]_2)$
2. If it is true, the user does not have the token. If It is false, the user has the token.

**How to know each term:**

1. $[B(s)]_1$ is the commitment stored in the contract through `submitCommitment` by a prover.
2. $[Q_B(w^i)]_1$ is given by the verifier as an input for `verifyInclusionProof`.
3. The index of a user $i$ can be known by a rainbow table trick. It calculates $[s]_{2}- [w^i]_2$ for every i and compare with the challenge input given in `verifyInclusionProof`.

### 2. High: [Wrong verifying key contract permutation length can be considered valid by `validateVKPermutationsLength`](https://github.com/zBlock-2/summa-solvency/issues/10)

By: **obatirou**

The function `validateVKPermutationsLength` is used to validate the number of permutations in the verifying key contract corresponds to the circuit used to generate proof for the number of cryptocurrencies the custodian committed to. The Summa contract assumes that permutations commitments begins at bytes `0x2e0` in the `vKContract` ([source](https://github.com/zBlock-2/summa-solvency/blob/fec83a747ead213261aecfaf4a01b43fff9731ee/contracts/src/Summa.sol#L151))
This can lead to wrongfully consider valid vkContracts with different permutation commitments from the circuit used by the custodian. A simple case would be a verifying contract with 2 fixed commitments but one permutation commitment less than the one being used. In this case, all others things being equals, the permutations commitments would not begin at bytes `0x2e0` but at `0x0320`. As the length of the bytes of the contracts are still equals due to one permutation in less, `validateVKPermutationsLength` would consider this vkcontract valid but in fact is not.

### 3. High: [Fake user Inclusion Proof verified in contract](https://github.com/zBlock-2/summa-solvency/issues/9)

By: **rkdud007**, **qpzm**

We managed to generate fake inclusion proof of a non-existing user, the user id is out of the bounds of the original CSV file. As all the balance cells’ values are zero, it might not be a critical issue in the CEX use case of proof of liability, but this becomes critical issue when Summa’s core logic is utilized in more [general application use cases](https://eprint.iacr.org/2021/1350.pdf) that inclusion itself plays an important role.

### 4. High: [Username overflow](https://github.com/zBlock-2/summa-solvency/issues/2)

By: **bbresearcher**

The same issue as on Version A of Summa still exists in Version B: https://github.com/zBlock-2/audit-report/blob/parsely-edits/versionA.md#1-high-possible-overflow-in-username-in-big_intify_username-combined-with-calling-big_uint_to_fp-and-guarantee-usernames-stays-inside-field

### Medium

### 1. Medium: [The return value of `GrandSumVerifier` should be tested](https://github.com/zBlock-2/summa-solvency/issues/4)

By: **qpzm**

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

### Low

### 1. Low: [Incorrect permutation length in `validateVKPermuationsLength`](https://github.com/zBlock-2/summa-solvency/issues/6)

By: **zeroqn**

The error arises from the range check lookup table. For instance, when the `balanceByteRange` is set to 9, we expect to have four 2^16 lookup tables plus an additional 2^8 or 2^16 lookup table. `balanceByteRange / 2` return 4, which is wrong.

**To Reproduce**: [change `balanceByteRange` to 9](https://github.com/summa-dev/summa-solvency/blob/v2/contracts/test/Summa.ts#L43)

**Expected behavior**: test should fail

### 2. Low: [CSV parsing allows duplicate crypto tokens](https://github.com/zBlock-2/summa-solvency/issues/5)

By: **bbresearcher**

The CSV parsing function and code allows for the CSD import to contain duplicate crypto tokens.
This was discussed and does not seem to have any security impact however it was decided to restrict columns to be unique.

**To Reproduce**:
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

**Expected behavior**: crypto token columns should be unique

### 3. Low: [The return value of `GrandSumVerifier` should be tested](https://github.com/zBlock-2/summa-solvency/issues/4)

By: **qpzm**

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

### 4. Low: [Range check tests are unreliable regarding the number of overflows](https://github.com/zBlock-2/summa-solvency/issues/3)

By: **qpzm**, **pia**

This test data has two entries which exceed 2^64 -1:

1. user0 balance0
2. user1 balance1

https://github.com/summa-dev/summa-solvency/blob/fec83a747ead213261aecfaf4a01b43fff9731ee/csv/entry_16_overflow.csv

The test result, however, shows 4 balances break the range check. For example, user2 balance0 is zero, but the test error contains "Perform range check on balance 0 of user 2".
https://github.com/summa-dev/summa-solvency/blob/fec83a747ead213261aecfaf4a01b43fff9731ee/prover/src/circuits/tests.rs#L453-L484

### Informational

### 1. Informational: [Security concerns in `Summa.sol`](https://github.com/zBlock-2/summa-solvency/issues/23)

By: **hrishibhat**

### 2. Informational: [Update memory locations in verifiers to save gas](https://github.com/zBlock-2/summa-solvency/issues/18)

By: **qpzm**, **rkdud007**

Verifier contracts specify memory addresses to load constants from the trusted setup.
It starts from 0x220 in [GrandSumVerifier](https://github.com/summa-dev/summa-solvency/blob/fec83a747ead213261aecfaf4a01b43fff9731ee/contracts/src/GrandSumVerifier.sol#L12) and from \0x200 in [InclusionVerifier](https://github.com/summa-dev/summa-solvency/blob/fec83a747ead213261aecfaf4a01b43fff9731ee/contracts/src/InclusionVerifier.sol#L11).
However, it can start from 0x180.

**Recommendation**:
The maximum memory size used in both verifiers is 0x180, so I recommend to save constants from memory 0x180.
The saved memory is 160 and 128 bytes respectively.

### 3. Informational: [Dynamic Errors not handled in Box of Errors for Function Results](https://github.com/zBlock-2/summa-solvency/issues/8)

By: **sachindkagrawal15**

When we use Box on function returning a result, the underlying error type is only known at runtime and not statically known. There are instances in code where Box is used but there is no handling of dynamic errors in the function.

### 4. Informational: [Use of unwrap in core files and improper error handling](https://github.com/zBlock-2/summa-solvency/issues/7)

By: **sachindkagrawal15**

Unwrap is usually a shortcut to bypass implementing proper error handling in Rust. Using Unwrap in production code can be dangerous and should be avoided as it crashes the application & hides actual bugs. So, This is highly undesirable in core files and is a Rust bad practice and proper error handling/transmission should be implemented.

By: **sachindkagrawal15**

## Final remarks

## Appendix

### A - Automated Analysis

#### 1. Halo2-analyzer

Halo2-analyzer / Korrekt employs a Satisfiability Modulo Theories (SMT) solver to try to find multiple satisfying models (i.e., private inputs) for a circuit and a (fixed) public input in order to determine if it is under-constrained. It works by collecting all the gates and regions using a custom implementation (`AnalyticLayouter`) of the `Layouter` trait. For every `region`, we have a set of enabled selectors and assigned cells (but with dummy values), likewise for every `gate`, we have a set of polynomials for each gate. This simple tool performs three checks :

- **Unused Gate -** Check that for every gate there exists a region in which it is not identically zero.
- **Unused Column -** Check that every column occurs in some polynomial.
- **Unconstrained Cell -** Check that for every assigned cell in the region, it occurs in a polynomial which is not identically zero over this region. This means that part of the witness is not constrained -- this is almost certainly a bug.

We used halo2-analyzer to search for unused gates, underconstrained cells and unused columns in the circuits. Here are the results :

### 1. Unused Gate

```bash
Finished analysis: 0 unused gates found.
```

### 2. Unused columns

```bash
Finished analysis: 5 unused columns found.
unused column: Column { index: 2, column_type: Advice }
unused column: Column { index: 3, column_type: Advice }
unused column: Column { index: 4, column_type: Advice }
unused column: Column { index: 5, column_type: Advice }
unused column: Column { index: 1, column_type: Advice }
```

### 3. Underconstrained Cells

```bash
Finished analysis: 12 unconstrained cells found.
unconstrained cell in "assign entries to the table" region: Column { index: 1, column_type: Advice } (rotation: 0) -- very likely a bug.
unconstrained cell in "assign entries to the table" region: Column { index: 0, column_type: Advice } (rotation: 0) -- very likely a bug.
unconstrained cell in "assign entries to the table" region: Column { index: 0, column_type: Advice } (rotation: 0) -- very likely a bug.
unconstrained cell in "assign entries to the table" region: Column { index: 1, column_type: Advice } (rotation: 0) -- very likely a bug.
unconstrained cell in "Perform range check on balance 0 of user 0" region: Column { index: 2, column_type: Advice } (rotation: 0) -- very likely a bug.
unconstrained cell in "Perform range check on balance 0 of user 0" region: Column { index: 5, column_type: Advice } (rotation: 0) -- very likely a bug.
unconstrained cell in "Perform range check on balance 0 of user 0" region: Column { index: 3, column_type: Advice } (rotation: 0) -- very likely a bug.
unconstrained cell in "Perform range check on balance 0 of user 0" region: Column { index: 4, column_type: Advice } (rotation: 0) -- very likely a bug.
unconstrained cell in "Perform range check on balance 0 of user 1" region: Column { index: 2, column_type: Advice } (rotation: 0) -- very likely a bug.
unconstrained cell in "Perform range check on balance 0 of user 1" region: Column { index: 3, column_type: Advice } (rotation: 0) -- very likely a bug.
unconstrained cell in "Perform range check on balance 0 of user 1" region: Column { index: 4, column_type: Advice } (rotation: 0) -- very likely a bug.
unconstrained cell in "Perform range check on balance 0 of user 1" region: Column { index: 5, column_type: Advice } (rotation: 0) -- very likely a bug.
```

#### 2. Polyexen-demo

Polyexen (Polynomial Expression Engine) transforms circuits designed with the Halo2 API into the Plonkish Arithmetization Format (Plaf). Plaf is designed to standardize plonkish circuits for reuse in components compatible with plonkish arithmetization. The current implementation supports serializing a Plaf circuit into two parts:

- A CSV file containing the fixed column values.
- A TOML file encompassing the rest of the circuit definition, including:
  - Lookup constraints
  - Gate constraints
  - Copy constraints

We used polyexen-demo to debug the assignments & double check the constraints. Here’s the output :

- Fixed Columns - [CSV](https://github.com/zBlock-2/summa-report/blob/main/ugc_fixed.csv)
- Lookup constraints - [ugc_lookups.toml](https://github.com/zBlock-2/summa-report/blob/main/ugc_lookups.toml)
- Gate constraints - [ugc_polys.toml](https://github.com/zBlock-2/summa-report/blob/main/ugc_polys.toml)
- Copy constraints - [ugc.toml](https://github.com/zBlock-2/summa-report/blob/main/ugc.toml)

#### 3. Highlighter

Highlighter is a python script which runs against a HALO2 (or Rust) project and will highlight any code that may need to be checked a bit closer

Highlighter works on a set of rules to look for error prone areas such as incorrect endianness, improper use of `unwrap`, finding `TODO` comments which might signify incomplete code, overflows & underflows. Here's the complete [report](https://github.com/zBlock-2/audit-report/blob/main/appendix/V2/Highlighter/output.md) of the findings by Highlighter.

#### 4. NPM Audit

`npm audit` scans your project's dependencies for known security vulnerabilities, reports them with severity levels, and suggests fixes. It helps keep your Node.js application secure by identifying and addressing potential risks in your packages. View the complete report of security vulnerabilities in the `contracts` package [here](https://github.com/zBlock-2/audit-report/blob/main/appendix/V2/npm-audit/output.md)

#### 5. Cargo Audit

`cargo audit` scans your Rust project's dependencies for known security vulnerabilities, reports them with severity levels, and suggests fixes. It helps keep your Rust application secure by identifying and addressing potential risks in your crates. View the complete report of security vulnerabilities in `prover` and `backend` [here](https://github.com/zBlock-2/audit-report/blob/main/appendix/V2/cargo-audit/output.md).

#### 6. Clippy

`clippy` is a linter for Rust that checks your code for common mistakes and style issues. It provides helpful suggestions to improve your code quality and maintainability. Using `clippy` helps ensure your Rust code is clean, efficient, and follows best practices. Here's the [report](https://github.com/zBlock-2/audit-report/blob/main/appendix/V2/clippy/output.md).


### [Methodology](#methodology)

The audit employed a blend of automated tools and manual examination conducted by the fellows and residents. Techniques included detailed code reviews, static and dynamic analysis, fuzzing, and penetration testing to ensure a thorough validation of the protocol’s security measures.

- 1. **Tool Integration:**
The audit utilized several specialized tools, each tailored to assess different aspects of the protocol:
    - **Halo2-analyzer**: Verified all circuit constraints.
    - **Polyexen-demo**: Standardized circuit formats for clarity and reusability.
    - **Highlighter**: Identified potential code issues needing closer examination.
    - **NPM and Cargo Audits**: Checked dependencies for known vulnerabilities.
    - **Clippy**: Ensured Rust code quality and best practices.
- 2. **Analytical Techniques:**
The audit encompassed both static and dynamic analyses to provide a comprehensive security assessment:
    - **Static Analysis**: Examined the source code for vulnerabilities without execution.
    - **Dynamic Analysis**: Tested the protocol in operation to identify runtime issues.
- 3. **Expert Review:**
We conducted in-depth manual reviews to evaluate complex components and integrations, providing a crucial layer of scrutiny beyond automated tools.
- 4. **Feedback and Improvements:**
An iterative feedback loop with the Summa’s development team allowed for the immediate addressing and re-evaluation of any issues found, ensuring all fixes were effectively implemented.
- 5. **Documentation:**
Each phase of the audit was thoroughly documented, with detailed reports on tool outputs, expert insights, and overall findings, culminating in a comprehensive final report that outlined vulnerabilities, impacts, and recommended actions.
