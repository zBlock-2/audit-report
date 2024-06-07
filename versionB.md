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
- [Methodology](#methodology)
- [Scope](#scope)
- [Automated testing](#automated-testing)
- [Fuzz testing](#fuzz-testing)
- [Findings Explanation](#findings-explanation)
- [Findings](#findings)
- [Final remarks](#final-remarks)
- [Recommendations](#recommendations)
- [Tool Configuration](#tool-configuration)
- [Appendix](#appendix)
## Protocol Summary

_For a hgih-level overview of the Summa protocol, [see this](./README.md#overview-of-the-summa-proof-of-solvency-protocol)_.

### Overview of the KZG-based Implementation of the Summa Proof of Solvency Protocol

The 2nd iteration of the Summa protocol aims to lower proving and verification costs while preserving the same privacy guarantees. The core primitive in this iteration of the protocol is a [cleverly](https://drive.google.com/drive/folders/1081DlyvigZykFaWmI3JCSknpnUEw0z67) [interpolated](https://drive.google.com/drive/folders/1k-8HDie_adueyjY0-TIhHaLawtT6p27d) polynomial that brings about performance efficiencies compared to the hashing-intensive MST in Version A. The overall flow of the protocol remains the same: the prover produces a proof of grand sum, and individual proofs of inclusion for each user. The reliance on large number of users verifying their inclusion proofs also remains, and the same privacy guarantees are also preserved.

A Lagrange basis polynomial is interpolated such that at `x=0` it evaluates to the grand sum of all liabilities (more precisely, the constant coefficient of the polynomial multiplied by the number of users = grand sum), while at `(w_j)^i` , where `w_j` is the `j-th` root of unity and `i` is a user-unique index, *it evaluates to the balance of the `i-th` user*. The construction takes advantage of the fact that the sum of roots of unity is zero in order to “encode” the desired evaluations into the polynomial.

The prover produces a commitment `C` to the polynomial and:

- An opening [proof](https://github.com/summa-dev/summa-solvency/blob/bd3a3d4d8fda6661e4eede00cf6176d66cb00858/prover/src/circuits/tests.rs#L167) at `x=0` to the public
- An opening proof at `(w_j)^i`  for the `i-th` user, who can can verify inclusion not only that the evaluation is correct but also that the commitment `C` they evaluated against is the same as the public commitment.

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


### 1. High: [The inclusion proof reveals whether a user holds a specific token](https://github.com/zBlock-2/summa-solvency/issues/21)
By: **qpzm**

A sniffer can see whether a user has a specific token because balance polynomials are unblinded.
A sniffer, for example, may be an web3 api endpoint who can see a user's inputs for `verifyInclusionProof`.

**Process**:
1. Assume $B(w^i) = 0$ and check $e([B(s)]_1 - [0]_1, [1]_2) == e([Q_B(w^i)]_1, [s]_2 - [w^i]_2)$
2. If it is true, the user does not have the token. If It is false, the user has the token.

**How to know each term:**
1. $[B(s)]_1$ is the commitment stored in the contract through `submitCommitment` by a prover.
3. $[Q_B(w^i)]_1$ is given by the verifier as an input for `verifyInclusionProof`.
4. The index of a user $i$ can be known by a rainbow table trick. It calculates $[s]_{2}- [w^i]_2$ for every i and compare with the challenge input given in `verifyInclusionProof`.

### 2. High: [Wrong verifying key contract permutation length can be considered valid by validateVKPermutationsLength](https://github.com/zBlock-2/summa-solvency/issues/10)
By: **obatirou**

The function `validateVKPermutationsLength` is used to validate the number of permutations in the verifying key contract corresponds to the circuit used to generate proof for the number of cryptocurrencies the custodian committed to. The Summa contract assumes that permutations commitments begins at bytes `0x2e0` in the `vKContract` ([source](https://github.com/zBlock-2/summa-solvency/blob/fec83a747ead213261aecfaf4a01b43fff9731ee/contracts/src/Summa.sol#L151))
This can lead to wrongfully consider valid vkContracts with different permutation commitments from the circuit used by the custodian. A simple case would be a verifying contract with 2 fixed commitments but one permutation commitment less than the one being used. In this case, all others things being equals, the permutations commitments would not begin at bytes `0x2e0` but at `0x0320`. As the length of the bytes of the contracts are still equals due to one permutation in less, `validateVKPermutationsLength` would consider this vkcontract valid but in fact is not.


### 3. High: [Fake user Inclusion Proof verified in contract](https://github.com/zBlock-2/summa-solvency/issues/9)
By: **rkdud007**, **qpzm**

We managed to generate fake inclusion proof of a non-existing user, the user id is out of the bounds of the original CSV file. As all the balance cells’ values are zero, it might not be a critical issue in the CEX use case of proof of liability, but this becomes critical issue when Summa’s core logic is utilized in more [general application use cases](https://eprint.iacr.org/2021/1350.pdf) that inclusion itself plays an important role.

### 4. High: [Same username overflow as reported in version A](https://github.com/zBlock-2/summa-solvency/issues/2)
By: **bbresearcher**

The same issue as on Version A of Summa still exists in Version B: https://github.com/zBlock-2/audit-report/blob/parsely-edits/versionA.md#1-high-possible-overflow-in-username-in-big_intify_username-combined-with-calling-big_uint_to_fp-and-guarantee-usernames-stays-inside-field

### 1. Medium: [The return value of `GrandSumVerifier` should be tested.](https://github.com/zBlock-2/summa-solvency/issues/4)
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


### 1. Low: [Incorrect permutation length in validateVKPermuationsLength](https://github.com/zBlock-2/summa-solvency/issues/6)
By: **zeroqn**

The error arises from the range check lookup table. For instance, when the `balanceByteRange` is set to 9, we expect to have four 2^16 lookup tables plus an additional 2^8 or 2^16 lookup table. `balanceByteRange / 2` return 4, which is wrong.

**To Reproduce**: [change `balanceByteRange` to 9](https://github.com/summa-dev/summa-solvency/blob/v2/contracts/test/Summa.ts#L43)


**Expected behavior**: test should fail

### 2. Low: [CSV parsing allows duplicate crypto tokens.](https://github.com/zBlock-2/summa-solvency/issues/5)
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

### 3. Low: [The return value of `GrandSumVerifier` should be tested.](https://github.com/zBlock-2/summa-solvency/issues/4)
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


### 4. Low: [Range check tests are unreliable regarding the number of overflows.](https://github.com/zBlock-2/summa-solvency/issues/3)
By: **qpzm**, **pia**

This test data has two entries which exceed 2^64 -1:
1. user0 balance0
2. user1 balance1

https://github.com/summa-dev/summa-solvency/blob/fec83a747ead213261aecfaf4a01b43fff9731ee/csv/entry_16_overflow.csv

The test result, however, shows 4 balances break the range check. For example, user2 balance0 is zero, but the test error contains "Perform range check on balance 0 of user 2".
https://github.com/summa-dev/summa-solvency/blob/fec83a747ead213261aecfaf4a01b43fff9731ee/prover/src/circuits/tests.rs#L453-L484


### 1. Informational: [Security concerns in `Summa.sol`](https://github.com/zBlock-2/summa-solvency/issues/23)
By: **hrishibhat**

### 2. Informational: [Update memory locations in verifiers to save gas](https://github.com/zBlock-2/summa-solvency/issues/18)
By: **qpzm**, **rkdud007**

Verifier contracts specify memory addresses to load constants from the trusted setup.
It starts from 0x220 in [GrandSumVerifier](https://github.com/summa-dev/summa-solvency/blob/fec83a747ead213261aecfaf4a01b43fff9731ee/contracts/src/GrandSumVerifier.sol#L12) and from \0x200 in [InclusionVerifier](
https://github.com/summa-dev/summa-solvency/blob/fec83a747ead213261aecfaf4a01b43fff9731ee/contracts/src/InclusionVerifier.sol#L11).
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

### 5. Informational: Automated tests for dependency vulnerabilities and code quality [1](https://github.com/zBlock-2/summa-solvency/issues/17) [2](https://github.com/zBlock-2/summa-solvency/issues/16) [3](https://github.com/zBlock-2/summa-solvency/issues/15) [4](https://github.com/zBlock-2/summa-solvency/issues/14) [5](https://github.com/zBlock-2/summa-solvency/issues/13)
By: **sachindkagrawal15**


## Final remarks
## Recommendations
## Tool Configuration
## Appendix
