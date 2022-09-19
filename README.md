# Standard Noir Example

This project demonstrates a basic example of how to prove and verify Noir circuits in typescript. 

## Requirements

- Noir is based upon [Rust](https://www.rust-lang.org/tools/install), and we will need to Noir's package manager `nargo` in order to compile our circuits. Further installation instructions for can be found [here](https://noir-lang.github.io/book/getting_started/install.html).
    - One of the most common issues installing `nargo` is the backend used for solving proofs. If you are having trouble, in the `nargo` crate's `Cargo.toml` you can replace the `aztec_backend` with this line: 
```
aztec_backend = { optional = true, git = "https://github.com/noir-lang/aztec_backend", rev = "91fcd5a41a63911cefbc2576efe678900644dffd", default-features = false, features = [
    "wasm-base",
] }
```
- The typescript tests and contracts live within a hardhat project, where we use [yarn](https://classic.yarnpkg.com/lang/en/docs/install/#mac-stable) as the package manager. 

## Development

Start by installing all the packages specified in the `package.json`

```shell
yarn install
```

After installing nargo it should be placed in our path and can be called from inside the `circuits` folder. We will then compile our circuit. This will generate an intermediate representation that is called the ACIR. More infomration on this can be found [here](https://noir-lang.github.io/book/acir.html).  

```shell
cd circuits/
nargo compile p
```

From this point all of our tests using just the typescript wrapper should pass. We use these three packages to interact with the ACIR. `@noir-lang/noir_wasm` to serialize the ACIR from file. `@noir-lang/aztec_backend` to serialize the solved witness or to solve the witness in typescript. Finally, `@noir-lang/barretenberg` to ultimately generate a proof and verify that proof.

If you were to run the tests right now though you would notice one test is failing. Currently, the Noir typescript wrappers need to add updated functionality for generating a verifier in Solidity. Currently only a proof generated by nargo will work for this Solidity verifier. 

Assuming you are stil in the `circuits/` folder run the commands below:

```shell
nargo prove p
nargo verify p
nargo contract
```

Make sure that the verify command returns `true` in the command line output. You can then move this verifier to the `contracts` folder in the root directory of this project. Note that if you make changes to the circuit, you must also generate a new verifier or else the verification will fail.

Try now running:

```shell
npx hardhat test
```

### Future work

- The contract command is being updated to match the backend that is used in the typescript wrapper. It will be possible for devs to generate the contract within typescript.
- Match the serialization of buffers in JS to what the Rust/C++ backends use to allow for more complex functionality such as hashes to be verified. Currently, if you want to add pedersen hash functionality that will be verified correctly you must write a Rust script that uses the complex functions within Noir's internal packages. 

- For example if you wanted to use pedersen in your circuit, you should start by making a Rust scripts folder. In `Cargo.toml` you would put a dependency to the `aztec_backend`
    
```
aztec_backend = { git = "https://github.com/noir-lang/aztec_backend", branch = "kw/wasm-module", default-features = false, features = ["wasm-base"] }
acvm = { git = "https://github.com/noir-lang/noir", features = ["bn254"] }
```
Then inside your script file you would import some Noir specific types. 
```
use aztec_backend::barretenberg_wasm::Barretenberg;
use acvm::FieldElement;
```

This can then be used to generate inputs that are generated by Noir's std lib inside of a given circuit.

```
let mut barretenberg = Barretenberg::new();

let test_field = FieldElement::from_hex("0x01").unwrap();
let test_field2 = FieldElement::from_hex("0x02").unwrap();
let test_field3 = FieldElement::from_hex("0x03").unwrap();
let test_field4 = FieldElement::from_hex("0x04").unwrap();
let salt = FieldElement::from_hex("0x32").unwrap();

let mastermind_hash = barretenberg.compress_many(vec![salt, test_field, test_field2,test_field3, test_field4]);

println!("mastermind_hash: {:?}", mastermind_hash.to_hex());
```
`mastermind_hash` would then be an input to the program. This is only valid though for programs that are proven and verified using `nargo`. This mismatch is being worked on and the typescript wrapper is being updated to allow all this functionality within typescript.
