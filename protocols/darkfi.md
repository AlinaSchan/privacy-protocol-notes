# darkfi

a layer one proof-of-work chain where every operation is a contract call carrying zero-knowledge
proofs. written against the darkfi repository at commit `a5514cb2` (2026-09-10). the longer notes
are in [darkfi-protocol-notes](https://github.com/alinaschanz/darkfi-protocol-notes).

## in its own words

"a new Layer 1 blockchain, designed with anonymity at the forefront" (`README.md`). the website
calls it "an L1 for anonymous engineering" (dark.fi, read 2026-09-15).

## status

alpha testnet; the node config has a mainnet section and the testnet guide says mainnet is not
active (`doc/src/testnet/node.md`). proof of work with randomx, merge-mineable with monero
(`doc/src/arch/consensus.md`, `doc/src/testnet/merge-mining.md`). the repository has been public
since 2020-08-20; agpl 3.0; canonical host git.dark.fi, a copy on github.

## public state

- block headers with height, timestamp, transactions root, state root, proof-of-work data, and a
  producer signature (`doc/src/arch/consensus.md`)
- every transaction's call tree: contract ids, payloads, proofs, signatures
  (`doc/src/spec/concepts.md`)
- per spent coin: nullifier, merkle root, value commitment, token commitment, spend hook, an
  encrypted user data field, the signature's public key (`src/contract/money/src/model/mod.rs`)
- per new coin: coin commitment, value and token commitments, encrypted note (same file)
- the fee of every transaction, in the native token (`src/contract/money/src/entrypoint/fee_v1.rs`)
- contract state trees, contract wasm and circuits, over rpc (`bin/darkfid/src/rpc/`)

## private state

values, token ids, sender and recipient keys, which output a spend consumes, memos
(`doc/src/arch/tx_lifetime.md`, `src/contract/money/proof/burn_v1.zk`). the dao's membership,
treasury and vote weights are hidden by design (`doc/src/start-here.md`, `doc/src/arch/dao.md`).

## identity assumptions

no accounts. a coin commits to its owner's public key; the key that signs a spend comes from a
separate secret (`burn_v1.zk`). block producers are identified by their signing key. viewing keys
for the money contract are listed as an open task, so there is no documented way to disclose a
wallet's history without its spending key (`doc/src/dev/contrib/contrib.md`).

## metadata exposure and network-level privacy

the project's own p2p library speaks tcp, tls, tor, i2p, nym, socks5, quic and unix sockets
(`src/net/transport/`). a peer learns your transport address, software version, clock and
advertised addresses from the version message (`src/net/message.rs`). the p2p layer adds no mixing;
hiding a node's address is delegated to the transport "subject to that transport's threat model"
(`doc/src/start-here.md`).

## programmability

contracts are wasm binaries with zk circuits written in zkas and run by a halo2 based virtual
machine; the three native contracts are money, dao and deployooor, the last for deploying others
(`src/contract/`, `doc/src/zkas/zkvm.md`). execution is split into read-only and write-only
sections, and the whole call tree is bundled by the sender (`doc/src/spec/concepts.md`).

## provenance and what can be verified

halo2 needs no trusted setup (`doc/src/zkas/zkvm.md`). the code, the book and the enhancement
proposals live in one repository. one audit is linked, by zksecurity, q1 2024
(`doc/src/start-here.md`). i did not check for reproducible builds.

## trust assumptions

proof-of-work miners for ordering; seed nodes under dark.fi for discovery (`doc/src/dep/0007.md`);
the chosen transport network for address privacy; the wallet's own node, since `drk` reads blocks
through a `darkfid` (`doc/src/arch/tooling.md`).

## open questions

how fees will price zk work; which features are live on the testnet at a given time; what a dao
vote reveals about its proposal; how transactions are relayed between peers. see the observable
state page of the longer notes.
