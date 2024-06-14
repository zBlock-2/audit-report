# zk_prover

```bash
1 Error vulnerability found :
Crate: shlex
Version: 1.2.0
Title: Multiple issues involving quote API
Date: 2024-01-21
ID: RUSTSEC-2024-0006
URL: https://rustsec.org/advisories/RUSTSEC-2024-0006
Solution: Upgrade to >=1.3.0
Dependency tree:
shlex 1.2.0
└── bindgen 0.66.1
└── c-kzg 0.1.1
├── revm-primitives 1.3.0
│ ├── revm-precompile 2.2.0
│ │ └── revm 3.5.0
│ │ └── halo2_solidity_verifier 0.1.0
│ │ └── summa-solvency 0.1.0
│ └── revm-interpreter 1.3.0
│ └── revm 3.5.0
└── revm-precompile 2.2.0

4 warnings details :
Crate: serde_cbor
Version: 0.11.2
Warning: unmaintained
Title: serde_cbor is unmaintained
Date: 2021-08-15
ID: RUSTSEC-2021-0127
URL: https://rustsec.org/advisories/RUSTSEC-2021-0127
Dependency tree:
serde_cbor 0.11.2
└── criterion 0.3.6
└── summa-solvency 0.1.0

Crate: atty
Version: 0.2.14
Warning: unsound
Title: Potential unaligned read
Date: 2021-07-04
ID: RUSTSEC-2021-0145
URL: https://rustsec.org/advisories/RUSTSEC-2021-0145
Dependency tree:
atty 0.2.14
└── criterion 0.3.6
└── summa-solvency 0.1.0

Crate: const-cstr
Version: 0.3.0
Warning: unsound
Title: const-cstr is Unmaintained
Date: 2023-03-12
ID: RUSTSEC-2023-0020
URL: https://rustsec.org/advisories/RUSTSEC-2023-0020
Dependency tree:
const-cstr 0.3.0
└── yeslogic-fontconfig-sys 3.2.0
└── font-kit 0.11.0
└── plotters 0.3.5
├── summa-solvency 0.1.0
├── halo2_proofs 0.2.0
│ ├── summa-solvency 0.1.0
│ ├── halo2_solidity_verifier 0.1.0
│ │ └── summa-solvency 0.1.0
│ └── halo2_gadgets 0.2.0
│ └── summa-solvency 0.1.0
└── criterion 0.3.6
└── summa-solvency 0.1.0

Crate: iana-time-zone
Version: 0.1.59
Warning: yanked
Dependency tree:
iana-time-zone 0.1.59
└── chrono 0.4.31
├── plotters 0.3.5
│ ├── summa-solvency 0.1.0
│ ├── halo2_proofs 0.2.0
│ │ ├── summa-solvency 0.1.0
│ │ ├── halo2_solidity_verifier 0.1.0
│ │ │ └── summa-solvency 0.1.0
│ │ └── halo2_gadgets 0.2.0
│ │ └── summa-solvency 0.1.0
│ └── criterion 0.3.6
│ └── summa-solvency 0.1.0
├── ethers-etherscan 2.0.12
│ └── ethers 2.0.11
│ └── summa-solvency 0.1.0
└── ethers-core 2.0.12
├── ethers-solc 2.0.12
│ ├── ethers-etherscan 2.0.12
│ └── ethers 2.0.11
├── ethers-signers 2.0.11
│ ├── ethers-middleware 2.0.11
│ │ └── ethers 2.0.11
│ └── ethers 2.0.11
├── ethers-providers 2.0.11
│ ├── ethers-middleware 2.0.11
│ ├── ethers-contract 2.0.11
│ │ ├── ethers-middleware 2.0.11
│ │ └── ethers 2.0.11
│ └── ethers 2.0.11
├── ethers-middleware 2.0.11
├── ethers-etherscan 2.0.12
├── ethers-contract-derive 2.0.12
│ └── ethers-contract 2.0.11
├── ethers-contract-abigen 2.0.12
│ ├── ethers-contract-derive 2.0.12
│ └── ethers-contract 2.0.11
├── ethers-contract 2.0.11
├── ethers-addressbook 2.0.12
│ └── ethers 2.0.11
└── ethers 2.0.11
```

# Backend

```bash
1 Error vulnerability found :
Crate: h2
Version: 0.3.20
Title: Resource exhaustion vulnerability in h2 may lead to Denial of Service (DoS)
Date: 2024-01-17
ID: RUSTSEC-2024-0003
URL: https://rustsec.org/advisories/RUSTSEC-2024-0003
Solution: Upgrade to ^0.3.24 OR >=0.4.2
Dependency tree:
h2 0.3.20
├── reqwest 0.11.18
│ ├── summa-backend 0.1.0
│ ├── ethers-providers 2.0.7
│ │ ├── ethers-middleware 2.0.7
│ │ │ └── ethers 2.0.7
│ │ │ ├── summa-solvency 0.1.0
│ │ │ │ └── summa-backend 0.1.0
│ │ │ └── summa-backend 0.1.0
│ │ ├── ethers-contract 2.0.7
│ │ │ ├── ethers-middleware 2.0.7
│ │ │ └── ethers 2.0.7
│ │ └── ethers 2.0.7
│ ├── ethers-middleware 2.0.7
│ └── ethers-etherscan 2.0.7
│ ├── ethers-middleware 2.0.7
│ └── ethers 2.0.7
└── hyper 0.14.27
├── reqwest 0.11.18
└── hyper-tls 0.5.0
└── reqwest 0.11.18

3 warnings details :

Crate: openssl
Version: 0.10.55
Warning: unsound
Title: openssl X509StoreRef::objects is unsound
Date: 2023-11-23
ID: RUSTSEC-2023-0072
URL: https://rustsec.org/advisories/RUSTSEC-2023-0072
Dependency tree:
openssl 0.10.55
└── native-tls 0.2.11
├── tokio-native-tls 0.3.1
│ ├── reqwest 0.11.18
│ │ ├── summa-backend 0.1.0
│ │ ├── ethers-providers 2.0.7
│ │ │ ├── ethers-middleware 2.0.7
│ │ │ │ └── ethers 2.0.7
│ │ │ │ ├── summa-solvency 0.1.0
│ │ │ │ │ └── summa-backend 0.1.0
│ │ │ │ └── summa-backend 0.1.0
│ │ │ ├── ethers-contract 2.0.7
│ │ │ │ ├── ethers-middleware 2.0.7
│ │ │ │ └── ethers 2.0.7
│ │ │ └── ethers 2.0.7
│ │ ├── ethers-middleware 2.0.7
│ │ └── ethers-etherscan 2.0.7
│ │ ├── ethers-middleware 2.0.7
│ │ └── ethers 2.0.7
│ └── hyper-tls 0.5.0
│ └── reqwest 0.11.18
├── reqwest 0.11.18
└── hyper-tls 0.5.0

Crate: ahash
Version: 0.8.3
Warning: yanked
Dependency tree:
ahash 0.8.3
└── hashbrown 0.13.2
└── revm-primitives 1.1.2
├── revm-precompile 2.0.3
│ └── revm 3.3.0
│ └── halo2_solidity_verifier 0.1.0
│ └── summa-solvency 0.1.0
│ └── summa-backend 0.1.0
└── revm-interpreter 1.1.2
└── revm 3.3.0

Crate: elliptic-curve
Version: 0.13.5
Warning: yanked
Dependency tree:
elliptic-curve 0.13.5
├── k256 0.13.1
│ ├── revm-precompile 2.0.3
│ │ └── revm 3.3.0
│ │ └── halo2_solidity_verifier 0.1.0
│ │ └── summa-solvency 0.1.0
│ │ └── summa-backend 0.1.0
│ ├── ethers-core 2.0.7
│ │ ├── ethers-solc 2.0.7
│ │ │ ├── ethers-etherscan 2.0.7
│ │ │ │ ├── ethers-middleware 2.0.7
│ │ │ │ │ └── ethers 2.0.7
│ │ │ │ │ ├── summa-solvency 0.1.0
│ │ │ │ │ └── summa-backend 0.1.0
│ │ │ │ └── ethers 2.0.7
│ │ │ └── ethers 2.0.7
│ │ ├── ethers-signers 2.0.7
│ │ │ ├── ethers-middleware 2.0.7
│ │ │ └── ethers 2.0.7
│ │ ├── ethers-providers 2.0.7
│ │ │ ├── ethers-middleware 2.0.7
│ │ │ ├── ethers-contract 2.0.7
│ │ │ │ ├── ethers-middleware 2.0.7
│ │ │ │ └── ethers 2.0.7
│ │ │ └── ethers 2.0.7
│ │ ├── ethers-middleware 2.0.7
│ │ ├── ethers-etherscan 2.0.7
│ │ ├── ethers-contract-derive 2.0.7
│ │ │ └── ethers-contract 2.0.7
│ │ ├── ethers-contract-abigen 2.0.7
│ │ │ ├── ethers-contract-derive 2.0.7
│ │ │ └── ethers-contract 2.0.7
│ │ ├── ethers-contract 2.0.7
│ │ ├── ethers-addressbook 2.0.7
│ │ │ └── ethers 2.0.7
│ │ └── ethers 2.0.7
│ ├── enr 0.8.1
│ │ └── ethers-providers 2.0.7
│ └── coins-bip32 0.8.3
│ ├── ethers-signers 2.0.7
│ └── coins-bip39 0.8.6
│ └── ethers-signers 2.0.7
├── ethers-signers 2.0.7
├── ethers-core 2.0.7
└── ecdsa 0.16.7
└── k256 0.13.1
```