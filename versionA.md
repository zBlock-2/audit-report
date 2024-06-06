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
- [Protocol Goals](#protocol-goals)
- [Scope](#scope)
- [Code Evaluation Matrix](#code-evaluation-matrix)
- [Automated testing](#automated-testing)
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


### 1. High: [Possible Overflow in username in big_intify_username combined with calling big_uint_to_fp](https://github.com/zBlock-2/summa-solvency-diffie/issues/16) and [Guarantee usernames stays inside field](https://github.com/zBlock-2/summa-solvency-schneier/issues/13)
By: **sebastiantf, bbresearcher**

**NB Need to give credit to Jin.s for the guidance to find the bug**
**Describe the bug**
A malicious prover could create usernames that overflow if two users have the same balance thus they can exclude one of the records from the data.

Checked the bug with the developers below is a response from the developers.
Copy/Paste from Discord chat
```text
Thank you for pointing that out. I understand the issue now.

Let's assume we have a username converted to 0x30644e72e131a029b85045b68181585d2833e84879b9709143e1f593f0000002, and another user with the identity 0x1, both of whom share the same user identity in a Merkle sum tree. This indicates a problem, as user identities in the Merkle sum tree should not be duplicated. You've made a great observation here.

In a worst-case scenario, two users share the same identity in the Merkle sum tree:
UserA: 0x30644e72e131a029b85045b68181585d2833e84879b9709143e1f593f0000002
UserB: 0x1
And both have the same amount of currencies. In this case, a malicious prover might exclude one of the elements from the tree and provide the same proof to different users with different usernames.
Although we add an assertion check for overflow, a malicious prover could bypass this. Therefore, it's important to inform users that converting their user identity over the modulus could lead to malicious activity.

From the perspective of the current implementation, this issue might not affect the generation of inclusion proofs for specific users, as the backend generates proofs using the user index rather than the user identity.

We should add a overflow check for username in on-chain verifier rather than rust implementation.
```
**Where is this code used**
https://github.com/summa-dev/summa-solvency/blob/master/zk_prover/src/circuits/merkle_sum_tree.rs#L252

https://github.com/summa-dev/summa-solvency/blob/master/zk_prover/src/merkle_sum_tree/tree.rs#L74

https://github.com/summa-dev/summa-solvency/blob/master/zk_prover/src/merkle_sum_tree/node.rs#L21



**Steps to recreate**
```rust
    let usr1 = big_intify_username("30644e72e131a029b85045b68181585d2833e84879b9709143e1f593f0000002");
    let ua = big_uint_to_fp(&usr1);
    println!("User1 value is : {:?}",ua);
    let usr2 = big_intify_username("1");
    let ub = big_uint_to_fp(&usr1);
    println!("User2 value is : {:?}",ub);
    if ua == ub {
        println!("The two user values are equal");
    }
```
Gives output of:
```text
User1 value is : 0x29fc300ca92b9374edb232c3567cceda058441c9c0c0d886ed5577f6f3ead4f2
User2 value is : 0x29fc300ca92b9374edb232c3567cceda058441c9c0c0d886ed5577f6f3ead4f2
The two user values are equal
```

**Expected behavior**
Usernames should be unique all the time


### 2. High: [Sum balance overflow](https://github.com/zBlock-2/summa-solvency-diffie/issues/10)
By: **zeroqn**

**Describe the bug**

There is no range check in the circuit for the sum of balances, which poses a risk of overflow.

Furthermore, since N_BYTES is not exposed in the contract, users must run `examples/gen_inclusion_verifier.rs` to obtain a warning message about the risk of overflow.


**To Reproduce**
Steps to reproduce the behavior:

`examples/gen_inclusion_proof.rs` set `N_BYTES` to 32 and change one of `MBlfbBGI`'s balance to `21888242871839275222246405745257275088548364400416034343698204186575808495617` in `csv/entry_16.csv`.

A valid proof is generated and pass verification.

**Expected behavior**
A clear and concise description of what you expected to happen.

Cannot generate valid proof

**Additional context**
Add any other context about the problem here.

Perhaps we could use a new lookup argument to achieve acceptable performance?

Lasso

https://github.com/privacy-scaling-explorations/halo2/issues/194
https://github.com/DoHoonKim8/halo2-lasso/pull/4


### 3. High: [Inconsistency in range checks](https://github.com/zBlock-2/summa-solvency-Turing/issues/14)
By: **Y5Yash**

**Background**
The circuit checks for all levels in the tree if the sibling node's balance (and two leaf balances) is less than `m = 2 ** (NBYTES * 8)`. In the first look, this suggests that the max balance for the immediate parent of the leaf nodes at level 1 would be `2 * m` and the parent at level 2 would be `3 * m`, ..., and the max balance at the root will be `(NLEVEL - 1) * m`.

**The inconsistency**
Let's consider two scenarios for a tree with just four leaves. Scenario 1: The user corresponding to Leaf 1 generates a proof. Scenario 2: The user corresponding to Leaf 3 generates a proof. The following diagram represents node balance range checks. A green node represents a direct check in the circuit, and a red node represents forced ranges.

![Summa_inconsistent_scenario_1](https://github.com/teddav/summa/assets/30973335/6723cf91-3c24-4282-aa45-94239289480c)
![Summa_inconsistent_scenario_2](https://github.com/teddav/summa/assets/30973335/ae7626fc-15cf-44d6-9d1b-094ba714d8fd)

Notice that depending on which leaf is generating the proof, the inner nodes have different ranges of values allowed. For all the cases to be consistent together, the root node can't be more than `2 * m` (because you can find cases in which either of the root's children are checked to have max value `m`).

**Expected behavior**
Root's max balance is `(NLEVEL - 1) * m` as can be inferred from circuits/contracts.



### 1. Low: [Mixed endian usage in code](https://github.com/zBlock-2/summa-solvency-diffie/issues/17)
By: **bbresearcher**

**Describe the bug**
In the code here https://github.com/summa-dev/summa-solvency/blob/master/zk_prover/src/merkle_sum_tree/utils/operation_helpers.rs#L5-L17 ,
In specific big_intify_username uses big-endian and fp_to_big_uint uses little-endian.

**Expected behavior**
The same endianness should be used throughout



### 1. Informational: [Range check uses lookup_any instead of lookup](https://github.com/zBlock-2/summa-solvency-schneier/issues/18)
By: **obatirou**

The range check uses function `lookup_any` which was introduced in PSE fork to allow dynamic lookup by using a table expression. The table used for the range check do not change and is always the same: values from `0 to 2^8-1.` Hence a dynamic lookup is not necessary in this context. Usage of `lookup` should be preferred.


### 2. Informational: [`InclusionVerifier.yul`not generated](https://github.com/zBlock-2/summa-solvency-schneier/issues/16)
By: **flyingnobita**

**Describe the bug**

It is mentioned in the Summa Book on the [summa-solvency page](https://summa.gitbook.io/summa-book/backend/summa-solvency) that `InclusionVerifier.sol` and `InclusionVerifier.yul` will be generated by the `gen_inclusion_verifier.rs` script. However, only `InclusionVerifier.sol` is generated.

> The script will generate a new `InclusionVerifier.sol` and `InclusionVerifier.yul` contracts in [`contracts/src`](https://github.com/summa-dev/summa-solvency/tree/master/contracts/src).

**To Reproduce**
Steps to reproduce the behavior:
1. `cd zk_prover`
2. `cargo run --release --example gen_inclusion_verifier`

**Expected behavior**
`InclusionVerifier.yul` contracts should be generated in [contracts/src](https://github.com/summa-dev/summa-solvency/tree/master/contracts/src).

**Additional context**
This is related to the `halo2_solidity_verifier` being used for generating the smart contracts.


### 3. Informational: [Improvement to public inputs in contract](https://github.com/zBlock-2/summa-solvency-schneier/issues/12)
By: **sebastiantf**

# Description

The [`publicInputs`](https://github.com/zBlock-2/summa-solvency-schneier/blob/main/contracts/src/Summa.sol#L193) input to the contract is taken as an array. But its not a homogenous array. The expected [public inputs](https://summa.gitbook.io/summa-book/circuits/merkle-sum-tree-inclusion#public-inputs-outputs) are: user leaf hash, MST root followed by root balances:
https://github.com/zBlock-2/summa-solvency-schneier/blob/95d63fe1a55935542810138aa5d8de7f50f4e94b/contracts/src/Summa.sol#L188-L197

Since they are not homogenous or not values that have the same meaning, it might be better DX/UX to have them as separate meaningful inputs and combine them into an array within the function before submitting them to the verifier. eg. how the zkDrops contract does it: https://github.com/a16z/zkdrops/blob/a4e58bdad8391ffc133c3643c449be5d18b69832/zkdrops-contracts/contracts/PrivateAirdrop.sol#L42-L46
<img width="737" alt="Screenshot 2024-02-27 at 12 22 25 PM" src="https://github.com/zBlock-2/summa-solvency-schneier/assets/36922376/45941ac9-605c-4e4f-806f-1476eacf18ac">


### 4. Informational: [Use only mapping for `addressOwnershipProofs`](https://github.com/zBlock-2/summa-solvency-schneier/issues/11)
By: **sebastiantf**

# Description

Currently both the array [`addressOwnershipProofs`](https://github.com/zBlock-2/summa-solvency-schneier/blob/95d63fe1a55935542810138aa5d8de7f50f4e94b/contracts/src/Summa.sol#L68) and the mapping [`_ownershipProofByAddress`](https://github.com/zBlock-2/summa-solvency-schneier/blob/95d63fe1a55935542810138aa5d8de7f50f4e94b/contracts/src/Summa.sol#L83C41-L83C65) are being used to track address ownership proofs. But the use of both seems unnecessary, inefficient and error-prone.

Didn't really notice any specific use-case being served by the array.

If there isn't really any use of the array, then I suppose we could remove the array, modify the mapping to store the `AddressOwnershipProof` struct and only use that. This is probably going to be a lot more efficient and simpler

### 5. Informational: [`Summa.sol` : Issue with `submitProofOfAddressOwnership()`](https://github.com/zBlock-2/summa-solvency-schneier/issues/7)
By: **zzzuhaibmohd**

**Describe the bug**

Issue#1: `submitProofOfAddressOwnership()` does not allow to resubmit `AddressOwnershipProof` twice

Currently the require check at [Summa.sol#L120](https://github.com/zBlock-2/summa-solvency-schneier/blob/95d63fe1a55935542810138aa5d8de7f50f4e94b/contracts/src/Summa.sol#L120) allows the `onlyOwner` to add the `AddressOwnershipProof` associated with the `cexAddress` only once. 

But imagine a scenario, wherein wrong signature or message was submitted during the first iteration, it that case, there is no way to update these values in the future. 

---
Issue#2: `proofIndex` generation does no take into account the name of the chain

The root cause of the issue can be slightly linked to the first issue, The hash calculation at [Summa.sol#L117](https://github.com/zBlock-2/summa-solvency-schneier/blob/95d63fe1a55935542810138aa5d8de7f50f4e94b/contracts/src/Summa.sol#L117) does not take into consideration the chain as the input during calculation. While this might not look an issue for EVM chains. But when considering a multi chain architecture, using at least two input for generating the hash is a good practice. Currently thinking of a practical impact due to this.

**To Reproduce**
Yet to update PoC

**Expected behavior**
Issue#1: `submitProofOfAddressOwnership()` does not allow to resubmit `AddressOwnershipProof` twice

I my opinion , since the `submitProofOfAddressOwnership()` is permissioned(can only be called by onlyOwner), updating the message and signature should be allowed.

---
Issue#2: `proofIndex` generation does no take into account the name of the chain

Make the following changes in the `addressHash` generation to generate a more unique hash

```diff
-- bytes32 addressHash = keccak256(
--     abi.encodePacked(_addressOwnershipProofs[i].cexAddress)
-- );
++
++ bytes32 addressHash = keccak256(
++     abi.encodePacked(_addressOwnershipProofs[i].cexAddress, _addressOwnershipProofs[i].chain)
++ );
```

**Additional context**
Add any other context about the problem here.


### 6. Informational: [`Summa.sol` : Ownable: Does not implement 2-Step-Process for transferring ownership](https://github.com/zBlock-2/summa-solvency-schneier/issues/6)
By: **zzzuhaibmohd**

**Describe the bug**
The contracts `Summa.sol` does not implement a 2-Step-Process for transferring ownership.
So ownership of the contract can easily be lost when making a mistake when transferring ownership.

While the probability if this happening is highly unlikely, it is better to follow best security measures.

**To Reproduce**
[Summa.sol#L7](https://github.com/zBlock-2/summa-solvency-schneier/blob/95d63fe1a55935542810138aa5d8de7f50f4e94b/contracts/src/Summa.sol#L7)

**Expected behavior**
Consider using the **Ownable2Step** contract from OZ (https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/Ownable2Step.sol) instead.

The way it works is there is a `transferOwnership` to transfer the ownership and `acceptOwnership` to accept the ownership. Refer to `Ownable2Step.sol` contract linked above.

**Additional context**
Add any other context about the problem here.


### 7. Informational: [Potential `Summa::submitCommitment()` Gas limits](https://github.com/zBlock-2/summa-solvency-schneier/issues/4)
By: **sebastiantf**

# Description

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

**Describe the bug**
There are hardcoded integers used in the construction of the Poseidon chips

**Expected behavior**
Either create named constants or document the values used in a comment

**Additional context**
https://github.com/summa-dev/summa-solvency/blob/master/zk_prover/src/circuits/merkle_sum_tree.rs#L237-L245


### 9. Informational: [Review of the `Summa.sol` smart contract.](https://github.com/zBlock-2/summa-solvency-diffie/issues/12)
By: **hrishibhat**

The Summa contract allows for centralized exchanges to prove to their users of the inclusion of their balances in the Merke Sum Tree. 
The Summa contract is currently setup with the configuration that includes:
- mstLevels the number of levels to be used in an exchange's MST
- currenciesCount number of cryptocurrencies supported. 
- balanceByteRange is the bytes used to represent the balance. 


The CEX Owner submits proof of address ownership through `submitProofOfAddressOwnership`. This is an optimistic proof that these addresses belong to the exchange and the verification is done off-chain due to the large computational requirement that may not be feasible on-chain. 

The CEX owner submits a commitment using `submitCommitment` about its liabilities in the form of a struct that uses the Merkle root, balances of the root, and the timestamp.

Once the commitment is submitted the user can then verify using the `verifyInclusionProof` function with relevant proof issued by the CEX and the public inputs against the exchange commitment for the respective timestamp that their balances are accurately represented in the CEX's Merkle tree. The verification is done by an external Inclusion verifier contract. 



### Observations and Security concerns:

- **No timestamp validation**:
```
    function submitCommitment(
        uint256 mstRoot,
        uint256[] memory rootBalances,
        Cryptocurrency[] memory cryptocurrencies,
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
        uint256 mstRoot,
        uint256[] memory rootBalances,
        Cryptocurrency[] memory cryptocurrencies,
        uint256 timestamp
    ) public onlyOwner {
        require(mstRoot != 0, "Invalid MST root");
+       require(timestamp < block.timestamp, "Cannot submit future commitment");
+        require(timestamp > lastSubmitted, "Incorrect timestamp");
        lastSubmitted = timestamp;
        ....
```

**mstLevels and currenciesCount are fixed**:
mstLevels and currenciesCount are currently set within the summa config inside the constructor. However this could limit flexibility if the number of accounts or currencies changes over time. Allow for dynamic resizing of `mstLevels` and `currenciesCount`. 
While this may require commitment versioning so as to not impact the previous versions of commitments when the mstLevels or currencies change. 

**No validation for mstLevels or currency length when adding commitments**:
Commitment submissions do check if the `cryptocurrencies` length matches with the configured range of `currenciesCount` allowing for any potential manipulations with these. Although this may not have a significant impact since the user could verify the currencies, also not all the currencies may need to be included for a commitment but a there can be a sanity check 
`require(cryptocurrencies.length <= currenciesCount, "Exceeds currency count");`
Additional check can also be added to make sure the mst levels resulting out of the `rootBalances`/`currenciesCount` does not exceed the configured `mstLevels`.
 



## Final remarks

## Recommendations
## Tool Configuration
## Appendix
### [range check fuzzing](https://github.com/zBlock-2/summa-solvency-Turing/issues/15)
By: **teddav**
### [Install (+run) halo2-analyzer](https://github.com/zBlock-2/summa-solvency-Turing/issues/12)
By: **teddav**
### [Security vulnerability analysis for dependencies of summa-backend package](https://github.com/zBlock-2/summa-solvency-Turing/issues/11)
By: **sachindkagrawal15**
### [Security vulnerability analysis for dependencies of contracts package](https://github.com/zBlock-2/summa-solvency-Turing/issues/10)
By: **sachindkagrawal15**
### [Security vulnerability analysis for dependencies of summa-solvency package](https://github.com/zBlock-2/summa-solvency-Turing/issues/9)
By: **sachindkagrawal15**
### [Clippy static Analysis results for summa_backend package](https://github.com/zBlock-2/summa-solvency-Turing/issues/8)
By: **sachindkagrawal15**
### [Clippy static Analysis results for summa_solvency package](https://github.com/zBlock-2/summa-solvency-Turing/issues/7)
By: **sachindkagrawal15**
### [Set up the Fuzz testing suite in summa](https://github.com/zBlock-2/summa-solvency-Turing/issues/2)
By: **0xpanicError**
