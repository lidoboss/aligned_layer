# Aligned

> [!CAUTION]
> To be used in testnet only.
 
## Table of Contents

- [Aligned](#aligned)
  - [Table of Contents](#table-of-contents)
  - [The Project](#the-project)
  - [Operator Guide](#operator-guide)
  - [Aligned Infrastructure Guide](#aligned-infrastructure-guide)
  - [How to use the testnet](#how-to-use-the-testnet)
  - [FAQ](#faq)
  - [What is the objective of Aligned?](#what-is-the-objective-of-aligned)

## The Project

Aligned works with EigenLayer to leverage Ethereum consensus mechanism for ZK proof verification. Working outside the EVM, this allows for cheap verification of any proving system. This enables the usage of cutting edge algorithms, that may use new techniques to prove even faster. Even more, proving systems that reduce the proving overhead and add verifier overhead, now become economically feasible to verify thanks to Aligned.

## Operator Guide

If you want to run an operator, check our [Operator Guide](./README_OPERATOR.md)

## Aligned Infrastructure Guide

If you are developing in Aligned, or want to run your own devnet, check our [Infrastructure Guide](./README_INFRASTRUCTURE.md)

## How to use the testnet

### Contract Information

Testnet contract is deployed on Holesky on the address:

 ```0x58F280BeBE9B34c9939C3C39e0890C81f163B623```

### How to use the testnet

Download and install Aligned to send proofs in the testnet:

```bash
curl -L https://raw.githubusercontent.com/yetanotherco/aligned_layer/main/batcher/aligned/install_aligned.sh | bash
```

Then run the ```source``` command that should appear in the shell

If you are experiencing issues, upgrade by running the same command.

The downloaded binaries require:

- MacOS Arm64 (M1 or higher)
- Linux x86 with GLIBC_2.32 or superior (For example, Ubuntu 22.04 or higher)

If you don't meet these requirements, clone the repository, install rust, and then run:

```bash
make uninstall_aligned
make install_aligned_compiling
```

### Try it!

We are going to download a proof previously generated, send it to Aligned, and retrieve the results from Ethereum Holesky testnet. Aligned is using EigenLayer to do a fast and cheap verification of more than one thousand proofs per second.

Download an example SP1 proof file with it's ELF file using:

```bash
curl -L https://raw.githubusercontent.com/yetanotherco/aligned_layer/main/batcher/aligned/get_proof_test_files.sh | bash
```

Send the proof with:

```bash
rm -rf ~/.aligned/aligned_verification_data/ &&
aligned submit \
--proving_system SP1 \
--proof ~/.aligned/test_files/sp1_fibonacci.proof \
--vm_program ~/.aligned/test_files/sp1_fibonacci-elf \
--aligned_verification_data_path ~/.aligned/aligned_verification_data \
--conn wss://batcher.alignedlayer.com
```

You should get a response like this:

```bash
[2024-06-17T22:06:03Z INFO  aligned] Proof submitted to aligned. See the batch in the explorer:
    https://explorer.alignedlayer.com/batches/0x8ea98526e48f72d4b49ad39902fb320020d3cf02e6506c444300eb3619db4c13
[2024-06-17T22:06:03Z INFO  aligned] Batch inclusion data written into /Users/maurofab/aligned_verification_data/8ea98526e48f72d4b49ad39902fb320020d3cf02e6506c444300eb3619db4c13_225.json
[2024-06-17T22:06:03Z INFO  aligned] All messages responded. Closing connection...
https://explorer.alignedlayer.com/batches/0x8ea98526e48f72d4b49ad39902fb320020d3cf02e6506c444300eb3619db4c13```
```

You can use the link to the explorer to check the status of your transaction. Then after three blocks, you can check if it has been verified with:

```bash
aligned verify-proof-onchain \
--aligned-verification-data ~/.aligned/aligned_verification_data/*.json \
--rpc https://ethereum-holesky-rpc.publicnode.com \
--chain holesky
```

You should get this result:

```bash
[2024-06-17T21:58:43Z INFO  aligned] Your proof was verified in Aligned and included in the batch!
```

If the proof wasn't verified you should get this result:

```bash
[2024-06-17T21:59:09Z INFO  aligned] Your proof was not included in the batch.
```

This is the same as running the following curl, with the proper CALL_DATA.

```bash
curl -H "Content-Type: application/json" \
    --data '{"jsonrpc":"2.0","method":"eth_call","id":1, "params":[{"to": "0x58F280BeBE9B34c9939C3C39e0890C81f163B623", "data": "<CALL_DATA>"}]}' \
    -X POST https://ethereum-holesky-rpc.publicnode.com
```

This returns a 0x1 if the proof and it's associated data is correct and verified in Aligned, and 0x0 if not.

For example, this a correct calldata for a verified proof:

```bash
curl -H "Content-Type: application/json" \
  --data '{"jsonrpc":"2.0","method":"eth_call","id":1,"params":[{"to": "0x58F280BeBE9B34c9939C3C39e0890C81f163B623", "data": "0xfa534dc0c181e470901eecf693bfa6f0e89e837dcf35700cdd91c210a0ce0660e86742080000000000000000000000000000000000000000000000000000000000000000836371a502bf5ad67be837b21fa99bc381f7e8124f02042ffb80fa7ce27bc8f6f39fd6e51aad88f6f4ce6ab8827279cfffb922660000000000000000000000007553cb14bff387c06e016cb3e7946e91d9fe44a54ad5d888ce8343ddb16116a700000000000000000000000000000000000000000000000000000000000000e0000000000000000000000000000000000000000000000000000000000000007600000000000000000000000000000000000000000000000000000000000001007b2f4966c3ab3e59d213eda057734df28c323055a2a02f50bd286585cc80128c967250f2b9ad990485338fd2d49e83f47917983f5566da551d4c32e9063ea5641d94b04bac222e06ea18cbb617d0d52c7007cc8f8b30c435b8b8101bdff0ea8482436acf251652f00397f4cefa0bb8eea1c8addb6cf2ca843004b89d80c7e1e41344fd2387535fe4afcaafde27b04543d993bbbc7286154044913e5bd65b86d7cc4d47a90132a95d9ffecb913b414ba2d2f0b1d7b826eb5025a27bcadcc0d94cb125c9c9d556eac08dd6b0f5f55f68afe699f3c529442dbf1b47e968b3705ee2e1be4acb884d184a139a390cb94e9e5806686605dc0a025269bc3afd990c8302"}]}' \
  -X POST https://ethereum-holesky-rpc.publicnode.com
```

To get the call data for yours, you can use the ```encode_verification_data.py```:

To use it, first clone then repository, then move to the repository folder, and install the dependencies with a python venv:

```bash
python3 -m venv .aligned_venv
source .aligned_venv/bin/activate
python3 -m pip install -r examples/verify/requirements.txt
```

Then:

```bash
python3 examples/verify/encode_verification_data.py --aligned-verification-data ~/.aligned/aligned_verification_data/*.json
```

If you want to verify your proof in your own contract, use a static call to the Aligned contract. You can use the following [Caller Contract](examples/verify/src/VerifyBatchInclusionCaller.sol) as an example. The code will look like this:

```solidity
(bool callWasSuccessfull, bytes memory proofIsIncluded) = targetContract.staticcall(
    abi.encodeWithSignature(
        "verifyBatchInclusion(bytes32,bytes32,bytes32,bytes20,bytes32,bytes,uint256)",
        proofCommitment,
        pubInputCommitment,
        provingSystemAuxDataCommitment,
        proofGeneratorAddr,
        batchMerkleRoot,
        merkleProof,
        verificationDataBatchIndex
    )
);
require(callWasSuccessfull, "static_call failed");
```

If you want to learn more about how to check if your proof was verified in aligned, 
check the [Guide](./examples/verify/README.md).

If you want to send more types of proofs, read our [send proofs guide](./README_SEND_PROOFS.md).

If you want to know more about Aligned, read our [docs](docs/README.md).

## FAQ

## What is the objective of Aligned?

Aligned’s mission is to extend Ethereum’s zero-knowledge capabilities. We are certain the zero-knowledge proofs will have a key role in the future of blockchains and computation. We don’t know what that future will look like, but we are certain it will be in Ethereum. The question we want to share is: If we are certain zero-knowledge proofs are the future of Ethereum but we are not certain which of the many possible zero-knowledge futures will win. How can we build an infrastructure for Ethereum to be compatible with any future zero-knowledge proving system?

### What is the throughput of Aligned?

Aligned runs the verifier’s code natively. The verification time depends on the proof system, program run, and public input. Generally, most verifiers can be run in the order of ms on consumer-end hardware. We can optimize the code for speed and leverage parallelization by running it natively. Taking 3 ms per proof, Aligned could verify 300 proofs per second and, using parallelization, over 10,000 proofs per second.

### How does the throughput of Aligned compare with Ethereum?

Ethereum runs on top of the EVM. Each block is limited to 30,000,000 gas. Since the most efficient proof systems take at least 250,000 gas, Ethereum can verify 120 proofs per block. Aligned runs the code natively and leverages parallelization, reaching 10,000 proofs in the same period.

### Is Aligned an Ethereum L2?

Aligned is related to Ethereum but is not an L2 since it does not produce blocks. It is a decentralized network of verifiers.

### Does Aligned compete with L2s?

No. Aligned is a decentralized network of verifiers and has proof aggregation. It does not produce blocks or generate proofs of execution. Aligned provides L2s with fast and cheap verification for the proofs they generate, reducing settlement costs and enhancing cross-chain interoperability with quick and cheap bridging.

### What are the costs for Aligned?

The costs depend on task creation, aggregated signature or proof verification, and reading the results. The cost C per proof by batching N proofs is roughly:

$$
  C =\frac{C_{task} + C_{verification}}{N} + C_{read}
$$

Batching 1024 proofs using Aligned’s fast mode can cost around 2,100 gas in Ethereum (for a gas price of 8 gwei/gas and ETH = $3000, $0.05). As a helpful comparison, a transaction in Ethereum costs 21,000 gas, so you get proof verification for 1/10th of the transaction cost!

### Why do you have a fast and slow mode?

The fast mode is designed to offer very cheap verification costs and low latency. It uses crypto-economic guarantees provided by restaking; costs can be as low as 2100 gas. The slow mode works with proof aggregation, with higher fees and latency, and achieves the complete security of Ethereum. We verify an aggregated BLS signature (around 113,000 gas) in the fast mode. We verify an aggregated proof (around 300,000 gas) in the slow mode.

### Why don’t you run Aligned on top of a virtual machine?

Running on a virtual machine adds complexity to the system and an additional abstraction layer. It can also reduce Aligned's throughput, which is needed to offer really fast and cheap verification.

### Why don’t you build Aligned on top of a rollup?

The main problem with settling on top of a rollup is that you still need confirmation in Ethereum, which adds latency to the process. Besides, most rollups are not fully decentralized, even if they were, not to the extent of Ethereum. Aligned also achieves an already low verification cost in Ethereum, so it would not be convenient to build Aligned on top of a rollup in terms of latency, costs, and decentralization.

An L2 needs to use the EVM to settle in Ethereum. This means that the proofs need to be efficiently verified in the EVM, and their data made available there.

The EVM is not designed for ZK Verification, so most verifications are expensive.

To solve this, for pairing-based cryptography, Ethereum has added a precompile for verifications using the curve BN254.

But technology changes fast. BN254 security was demonstrated to be around 100 bits instead of the expected 128. Fast Starks need efficient hashing for fields. Which is the best field? Mersenne’s? Goldilocks? Binary fields? What about the sumcheck protocol? Is Jolt the endgame? Or is GKR going to be faster?

The amount of progress in the field is big, and nobody can predict the endgame.

Even more, it would be naive to think that only one optimized prover will exist in the future. In the world of ZK, as in many others, there are trade-offs and systems that solve different problems.

Maybe we want faster proving and don't care about proof size. Maybe we want the fastest proof verification and smallest size and can do more work on the prover. The system may be optimized to prove Keccak really fast. Or we can skip the traditional hashes altogether and just optimize for Poseidon, Rescue, or one hash not created yet.

Aligned solves all of this. No matter how or what you want to prove, it can be verified efficiently here while still inheriting the security of Ethereum as other L2s.

### Is Aligned an aggregation layer?

Aligned provides proof aggregation as part of its slow mode, a feature shared with all aggregation layers. However, Aligned offers a unique fast mode designed to provide cheap and low-latency proof verification, leveraging the power of restaking. Aligned is a decentralized network designed to verify zero-knowledge proofs and uses recursive proof aggregation as one of its tools.

### What proof systems do you support?

Aligned is designed to support any proof system. Currently supported ones are Groth 16 and Plonk (gnark), SP1, Halo 2 (IPA and KZG)

### How hard is it to add new proof systems?

Aligned is designed to make adding new proof systems easy. The only thing needed is the verifier function, which is written in a high-level language like Rust. For example, we could integrate Jolt into one of our testnets just a few hours after it was released.

### What are BLS signatures?

[Boneh-Lynn-Shacham](https://en.wikipedia.org/wiki/BLS_digital_signature) is a cryptographic signature that allows a user to verify that a signer is authentic. It relies on elliptic curve pairings and is used by Ethereum due to its aggregation properties.

### How does Aligned work?

The flow for fast verification is as follows:

1. The user uses a provided CLI or SDK to send one proof or many to the batcher, and waits (Alternatively, the user can run a batcher or interact directly with Ethereum)
2. The batcher accumulates proofs of many users for a small number of blocks (typically 1-3).
3. The batcher creates a Merkle Tree with commitments to all the data submitted by users, uploads the proofs to the Data Service, and creates the verification task in the ServiceManager.
4. The operators, using the data in Ethereum, download the proofs from the DataService. They then verify that the Merkle root is equal to the one in Ethereum, and verify all the proofs.
5. If the proofs are valid, they sign the root and send this to the BLS signature aggregator.
6. The signature aggregator accumulates the signed responses until reaching the quorum, then sends the aggregated signature to Ethereum.
7. Ethereum verifies the aggregated signatures and changes the state of the batch to verified.

### What is restaking?

EigenLayer introduced the concept of Restaking. It allows Ethereum’s validators to impose additional slashing conditions on their staked ETH to participate in Actively Validated Services (AVS) and earn additional rewards. This creates a marketplace where applications can rent Ethereum's trust without competing for blockspace. Aligned is an example of an AVS.

### How can I verify proofs in Aligned?

You can verify proofs in Aligned using our CLI.

### Can you provide an estimate of Aligned’s savings?

In Ethereum (does not include access cost):

- Groth 16 proofs: 250,000 gas
- Plonk/KZG proofs: >300,000 gas
- STARKs: >1,000,000 gas
- Binius/Jolt: too expensive to run!

In Aligned, fast mode:

- Just one proof (any!): 120,000 gas
- Batching 1024 proofs: 120 gas + reading cost

It’s over 99% savings!

### I want to verify just one proof. Can I use Aligned for cheap and fast verification?

Yes!

### Is Aligned open-source?

Yes!

### What are the goals of Aligned?

Aligned is an infrastructure that offers fast and cheap verification for zero-knowledge and validity proofs. It can take any proof system and verify it cheaply and fast.

This means that what Aligned wants to achieve is to allow anyone to build zk applications. This can only be achieved by:

- Reducing operational costs when maintaining a zk application -> anyone can afford to build zk apps.
- Offering more options so developers can choose how they want to build their protocols -> everyone can choose their tools.
- Offer the latest zk that allows anyone to build zk applications by just proving rust -> anyone can code a zk application.

### What’s the role of Aligned in Ethereum?

Aligned’s role is to help advance the adoption of zero-knowledge proofs in Ethereum, increase verification throughput, and reduce on-chain verification time and costs. Aligned can easily incorporate proof systems without any further changes in Ethereum. In a more straightforward analogy, Aligned is like a GPU for Ethereum.

### What is proof recursion?

Zero-knowledge proofs let you generate proofs that show the correct execution of programs. If a program is the verification of a proof, then we will be getting a proof that we verified the proof and the result was valid. The validity of the second proof implies the validity of the original proof. This is the idea behind proof recursion, and it can be used with two main goals:

1. Convert one proof type to another (for example, a STARK proof to a Plonk proof) either to reduce the proof size, have efficient recursion, or because the proof system cannot be verified where we want.
2. Proof aggregation: if we have to verify N proofs on-chain, we can generate a single proof that we verified the N proofs off-chain and just check the single proof in Ethereum.

Proof recursion is the primary tool of Aligned’s slow mode.

### What are the use cases of Aligned?

Among the possible use cases of Aligned, we have:

Soft finality for Rollups and Appchains, fast bridging, new settlement layers (use Aligned + EigenDA) for Rollups and Intent-based systems, P2P protocols based on SNARKs such as payment systems and social networks, alternative L1s interoperable with Ethereum, Verifiable Machine Learning, cheap verification and interoperability for Identity Protocols, ZK Oracles, new credential protocols such as zkTLS based systems, ZK Coprocessor, encrypted Mempools using SNARKs to show the correctness of the encryption, protocols against misinformation and fake news, and on-chain gaming.

### Why build Aligned on top of Ethereum?

Ethereum is the most decentralized and most significant source of liquidity in the crypto ecosystem. We believe it is the most ambitious and long-term project on the internet. Aligned is being built to help Ethereum achieve its highest potential, and we believe this is only possible through validity/zero-knowledge proofs.

### Why EigenLayer?

We believe Ethereum is the best settlement layer, and zero-knowledge will play a key role in helping it become the settlement layer of the internet. We want to build a verification layer that helps Ethereum achieve this goal. This layer needs to have a decentralized group of validators that will just re-execute the verification of different proofs, but how can we build such a decentralized network that will help Ethereum? Creating a new L1 doesn’t benefit Ethereum because it will add new trust assumptions to the Ethereum protocols relying on it. So, if we must have:

1. A decentralized network of verifiers
2. A similar economic security level that can be easily measured in Ethereum
3. Part of the Ethereum ecosystem
4. Flexible enough to support many current and future proving systems

### How does it compare to the Polygon aggregation layer?

Aligned is just a network of decentralized verifiers renting security from Ethereum. On the other hand, the Polygon aggregation layer, in essence, is a rollup verifying multiple proofs. That is not the case for Aligned, which just executes a rust binary from different verifiers directly in multiple Ethereum validators.

### Why do we need a ZK verification layer?

Verifiable computation allows developers to build applications that help Ethereum scale or even create applications that were not possible before, with enhanced privacy properties. We believe the future of Ethereum will be shaped by zero-knowledge proofs and help it increase its capabilities.
