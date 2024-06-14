# zk_prover

```bash
warning: very complex type used. Consider factoring parts into type definitions
--> src/chips/merkle_sum_tree.rs:113:10
|
113 | ...-> Result<(AssignedCell<Fp, Fp>, AssignedCell<Fp, Fp>), Error...
| ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
|
= help: for further information visit https://rust-lang.github.io/rust-clippy/master/index.html#type_complexity
= note: #[warn(clippy::type_complexity)] on by default

warning: very complex type used. Consider factoring parts into type definitions
--> src/chips/range/tests.rs:58:10
|
58 | ) -> Result<
| __________^
59 | | (
60 | | AssignedCell<Fp, Fp>,
61 | | AssignedCell<Fp, Fp>,
... |
64 | | Error,
65 | | > {
| |_____^
|
= help: for further information visit https://rust-lang.github.io/rust-clippy/master/index.html#type_complexity

warning: unneeded late initialization
--> src/circuits/merkle_sum_tree.rs:304:13
|
304 | ... let sibling_hash: AssignedCell<Fp, Fp>; // hash of the si...
| ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
|
= help: for further information visit https://rust-lang.github.io/rust-clippy/master/index.html#needless_late_init
= note: #[warn(clippy::needless_late_init)] on by default
help: declare sibling_hash here
|
308 | let sibling_hash: AssignedCell<Fp, Fp> = if level == 0 {
| ++++++++++++++++++++++++++++++++++++++++
help: remove the assignments from the branches
|
371 ~ computed_sibling_hash
372 | }
...
439 |
440 ~ computed_sibling_hash
|

warning: very complex type used. Consider factoring parts into type definitions
--> src/circuits/utils.rs:41:6
|
41 | ) -> Result<
| _____^
42 | | (
43 | | ParamsKZG,
44 | | ProvingKey,
... |
47 | | &'static str,
48 | | > {
| |^
|
= help: for further information visit https://rust-lang.github.io/rust-clippy/master/index.html#type_complexity
```