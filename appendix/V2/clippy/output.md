# backend

```bash
7 warnings as below:

warning: this expression creates a reference which is immediately dereferenced by the compiler
--> src/apis/round.rs:70:21
|
70 | signer: &signer,
| ^^^^^^^ help: change this to: signer
|
= help: for further information visit https://rust-lang.github.io/rust-clippy/master/index.html#needless_borrow
= note: #[warn(clippy::needless_borrow)] on by default

warning: single-character string constant used as pattern
--> src/apis/round.rs:140:50
|
140 | let parts: Vec<&str> = params_path.split("-").collect();
| ^^^ help: try using a char instead: '-'
|
= help: for further information visit https://rust-lang.github.io/rust-clippy/master/index.html#single_char_pattern
= note: #[warn(clippy::single_char_pattern)] on by default

warning: useless conversion to the same type: ethers::types::Bytes
--> src/contracts/generated/summa_contract.rs:14041:17
|
14041 | SUMMA_BYTECODE.clone().into(),
| ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ help: consider removing .into(): SUMMA_BYTECODE.clone()
|
= help: for further information visit https://rust-lang.github.io/rust-clippy/master/index.html#useless_conversion
= note: #[warn(clippy::useless_conversion)] on by default

warning: module has the same name as its containing module
--> src/contracts/generated/inclusion_verifier.rs:12:1
|
12 | / pub mod inclusion_verifier {
13 | | #[rustfmt::skip]
14 | | const __ABI: &str = "[{"inputs":[{"internalType":"bytes","name":"proof","type":"byt...
15 | | ///The parsed JSON ABI of the contract.
... |
38882 | | pub struct VerifyProofReturn(pub bool);
38883 | | }
| |_^
|
= help: for further information visit https://rust-lang.github.io/rust-clippy/master/index.html#module_inception
= note: #[warn(clippy::module_inception)] on by default

warning: useless conversion to the same type: ethers::types::Bytes
--> src/contracts/generated/inclusion_verifier.rs:38831:17
|
38831 | INCLUSIONVERIFIER_BYTECODE.clone().into(),
| ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ help: consider removing .into(): INCLUSIONVERIFIER_BYTECODE.clone()
|
= help: for further information visit https://rust-lang.github.io/rust-clippy/master/index.html#useless_conversion

warning: module has the same name as its containing module
--> src/contracts/mock/mock_erc20.rs:12:1
|
12 | / pub mod mock_erc20 {
13 | | #[rustfmt::skip]
14 | | const __ABI: &str = "[{"inputs":[],"stateMutability":"nonpayable","type":"constructor",...
15 | | ///The parsed JSON ABI of the contract.
... |
7226 | | pub struct TransferFromReturn(pub bool);
7227 | | }
| |_^
|
= help: for further information visit https://rust-lang.github.io/rust-clippy/master/index.html#module_inception

warning: useless conversion to the same type: ethers::types::Bytes
--> src/contracts/mock/mock_erc20.rs:6333:17
|
6333 | MOCKERC20_BYTECODE.clone().into(),
| ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ help: consider removing .into(): MOCKERC20_BYTECODE.clone()
|
= help: for further information visit https://rust-lang.github.io/rust-clippy/master/index.html#useless_conversion
```