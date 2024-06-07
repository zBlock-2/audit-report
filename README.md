# Summa PSE Audit Report by yAcademy

A collective of fellows and yAcademy ZK Resident auditors engaged with the Summa team over a 14-week period to perform a security review of the Summa protocol in both of its variants. The engagement involved live protocol and codebase overview sessions, office hour sessions, async discussions, theoretical and implementation puzzles and challenges in the form of bounties, and other buzzing activities throughout the this 2nd ZK Auditing fellowship by yAcademy.


- [Report for the MST iteration of the Summa protocol](./versionA.md)
- [Report for the KZG iteration of the Summa protocol](./versionB.md)

There is a [dedicated page](https://yacademy.dev) containing all outputs from this fellowship: reports, tooling, sessions, and highlights. Should you be an aspiring ZK auditor and want to simulate going through the fellowship, you can post any questions or difficulties you encounter in the public channels dedicated to [zero-knowledge discussions](https://discord.com/channels/877252171983360072/1106224054358261820) and our residents, alumni fellows, and the community at large can provide help. You can also [apply](https://yacademy.dev/fellowships) to be considered for future fellowships.


### Overview of the Summa Proof of Solvency Protocol

Blockchain technology can facilitate uncensorable and self-soverign control of one’s assets. Users nonetheless regularly transfer ownership of their assets to centralized entities for various needs, chief among which is trading of cryptocurrencies and the on/off-ramping of fiat. Such centralized entities may themselves and in turn transfer ownership to secondary specialized custodian entities or other service providers such as investment funds. Unfortunately, such centralized control and handling of user funds can lead to catastrophic situations such as the mishandling of private keys which may lead to internal or external hacks, or the outright misuse of funds for trading or for use as collateral to access capital -which can result in liquidations in extreme market conditions.

From the users point of view, they only see a promise from the centralized entity that they hold their funds. But this only represents entries in an accounting database, and may or may not reflect the state of wallets that are under the control of the centralized entity. The perennial question is: are all user funds available and liquid for immediate withdrawal at a given moment in time?

Summa takes an approach that focuses on binding the custodian to a certain claim about the sum of their liabilities to their users, and subsequently leveraging zero-knowledge and cryptographic primitives to prove that the assets under their control are equal or exceed that sum of liabilities. In other words, rather than focusing on proving reserves, as in "we the entity control the private key(s) of wallets holding the pooled deposits of users", Summa focuses on binding liabilities, as in "we the entity prove to each user that their balance is included in calculating a grand sum of all liabilities, and we prove control of wallets that contain funds equal or exceeding that aggregated balance of liabilities".

Summa’s 2-sided mechanism that overall provides a proof of solvency of an entity provides two useful proofs:

(a) **Proof of grand sums**: a single proof to the public from the centralized entity that it controls the private keys of wallets, the sum of each asset in which, sum up to a total that is greater than or equal to a *claimed* total sum of liabilities to its users in that asset.

(b) **Inclusion proofs**: Multiple proofs to users, one for each user, that their exact balances were included in the calculation of the grand sum. The more users verify their individual proof of inclusion of their exact balances of each asset (a proof which is cryptographically tied to the overall proof in (a)), the more confidence there is that the *claimed* total of liabilities used in (a) was truthful, thereby proving the solvency of the entity overall.

The more users verify their proof of inclusion in (b) the more trust the public at large can put in the proof of grand sums in (a). A custodian may incentivise wide verification of inclusion by users through the use of lottery where in each round of verification, say weekly or monthly, certain users are selected randomly to win a monetary reward.

![summa workflow](https://github.com/zBlock-2/audit-report/blob/main/assets/summa-workflow.png?raw=true)

Figure 1: General flow of the Summa protocol in both variants, *credit: [Enrico - Summa tech lead](https://docs.google.com/presentation/d/1xUcH8geMz6I1iD9Jx0kWsIZvUcVlii5Us3mM4Mb3HNg/edit#slide=id.p3)*

The proof in (a) further represents a trap-door commitment vis-a-vis the user who will verify their individual inclusion proofs against it. The zkSNARKs bring two benefits:

- **Privacy** against the leakage of user data. In verifying proofs of grand sums, the public input are the leaf and root hashes.
- **Validity** of the computation of aggregated balances.

The core of Summa protocol has been implemented in two variants that use different cryptographic primitives to generate the aforementioned proofs. However, the overall flow of the protocol (Figure 1) and assumptions and guarantees on security and privacy remain the same in both variants.

### [Overview of the MST-based implementation of the Summa protocol](./versionA.md#protocol-summary)

### [Overview of the KZG-based implementation of the Summa protocol](./versionB.md#protocol-summary)
