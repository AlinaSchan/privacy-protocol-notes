# anoma

an architecture for intent-centric applications built on a resource model, with a specification,
a reference node in elixir, a shielded resource machine in rust on risc zero, and protocol adapters
that settle on existing evm chains. written against anoma/anoma at commit `5bcbf848` (2026-04-09,
branch `base`) and anoma/nspec at `e850e7b2` (2025-06-03, tag v0.2.0), plus the arm-risc0 and
pa-evm repositories as read on github on 2026-09-15. citation key: `nspec` is `docs/` in the
specification repository (specs.anoma.net), `arm` is `arm/src` in arm-risc0, `pa` is
`contracts/src/ProtocolAdapter.sol` in pa-evm, `docs.anoma.net` pages by slug.

## in its own words

"anoma is a distributed operating system for intent-centric applications" (`nspec/index.md`). the
same page's front matter calls the document "a work-in-progress specification". the website says
"the distributed os that puts you in control" and names three parts: anoma dos, the anoma resource
machine "installed on existing blockchains via protocol adapters", and "a novel consensus system"
(anoma.net, read 2026-09-15).

## status

what is live, per the project's own blog and explorer: protocol adapter contracts on ethereum
mainnet since 2025-11-18, then arbitrum, base, optimism and bnb chain; the explorer indexes nine
chains at contracts version 1.1.0; anomapay v1 on ethereum and bnb since june 2026; the xan token
on ethereum since september 2025 (official blog posts, explorer.anoma.net/contracts). the roadmap
marks the protocol phase "galileo" live and the phases with controllers and native consensus not
live (anoma.net/roadmap).

the elixir node reached v1.0.0 on 2026-04-09, when the umbrella repository was broken into five
repositories and 31,000 lines left this one (git log, commit `103f6214`). the specification's
latest tag is v0.2.0 (2025-06-03). the whitepaper pdf says "pre-release, august 21, 2022" and was
built on 2025-08-07; its repository readme calls it "a working draft".

## the model

an intent is "a signed message that describes a partial state transition"; at the architecture
level intents are "opaque bytestrings"; solvers are permissionless (whitepaper, sections 5.2 to
5.4). in the spec's terms, users submit unbalanced resource machine transactions, solvers output
balanced ones, an ordering step follows, and an executor "verifies and executes the transactions in
the determined order" (`nspec/arch/system/state/resource_machine/index.juvix.md`).

a resource has eight fields: logic reference, label reference, value reference, quantity, an
ephemeral flag, nonce, nullifier key commitment and a random seed; resources "are immutable, they
can be created once and consumed once"; state is the set of active resources. creating a resource
publishes a commitment, consuming it publishes a nullifier. a transaction carries three kinds of
proof: a logic proof per resource, a compliance proof per unit of consumed and created resources,
and a delta proof that the quantities balance; "balance is the criterion of a transaction's
completeness" (`nspec/.../resource/index.juvix.md`, `.../compliance_unit/compliance_proof.juvix.md`,
`.../transaction/delta_proof.juvix.md`; the same eight fields in `arm/resource.rs`).

applications are resource logics plus transaction functions plus projection functions, and "the
arm is unaware of the existence of applications" (`nspec/.../notes/applications.juvix.md`). the spec
says the machine is "significantly inspired by the zcash protocol" and plays a role "similar to the
role of the ethereum virtual machine", processing resources instead of instructions.

## public state

on an evm chain with a protocol adapter: every `execute` call to the adapter, the commitments and
nullifiers it stores, application data blobs emitted as events, the number of tags per transaction
and of compliance units, whether the adapter is emergency-stopped, and the erc-20 movements into
and out of the forwarders, which are ordinary host-chain transfers ("shielding/unshielding
addresses via permit2" in anomapay's own privacy page) (`pa`, `nspec/arch/integrations/adapters/
evm.md`, docs.anoma.net anoma-pay-privacy-guarantees). the compliance instance carries the outer
logic verifying key, so which logic a resource uses is public on the deployed circuit; the evm page
says the deployed circuit brings "the loss of function privacy" (`nspec/.../compliance_proof.juvix.md`,
`evm.md`).

## private state

in a shielded instantiation: the resource objects (label and value preimages, quantity, nonce),
nullifier keys and merkle paths are witness data (`nspec/.../compliance_proof.juvix.md`). anomapay
hides sender, amount and receiver on chain; payloads are encrypted with aes-256-gcm to the owner's
key (docs.anoma.net anoma-pay-privacy-guarantees, `arm_gadgets/src/encryption.rs`). the spec calls a
machine shielded only "if it achieves both data and function privacy"; function privacy needs a
recursive "proof of logic proof validity", and the arm-risc0 readme marks the shielded machine
specification "tbd" (`nspec/.../index.juvix.md`, `.../action/index.juvix.md`, arm-risc0 readme).

## identity assumptions

a "knowledge-based identity interface": an identity is defined by knowing a secret; internal
identity signs and decrypts, external identity verifies and encrypts; relations `signsFor` and
`readsFor` carry evidence, and a special identity "true" can be signed for and read by anyone
(`nspec/arch/system/identity/identity.juvix.md`). a resource commits to its nullifier key, which
"shouldn't be known to the resource creator" (`nspec/.../roles_and_requirements.juvix.md`).
anomapay derives an authority key, a nullifier key, an encryption key and a discovery key from one
seed or from an evm wallet signature (docs.anoma.net key-management).

## metadata exposure and network-level privacy

the spec describes a network subsystem with pub/sub, node adverts that carry a public key,
addresses, version and signature, and transport engines "such as quic or tls"
(`nspec/arch/node/engines/net_registry_messages.juvix.md`, `.../transport_protocol.juvix.md`). the
node code speaks grpc on port 50051 and has no quic or tls engine (`lib/node/transport/`,
`config/config.exs`). in production, anomapay routes a signed intent through a transaction service
and a proving service that "receive your witness data", and the transaction service submits to the
adapter and pays the gas; the trust model page says the proving operator sees the unencrypted
witness "temporarily" and runs kyc and aml checks (docs.anoma.net anoma-pay-architecture,
anoma-pay-trust-model). whether intents are gossiped between nodes anywhere in production is not
clear from the public material.

## programmability

resource logics are the programs. on the risc zero path they are rust guest programs compiled for
the zkvm, with examples for a counter, a token transfer and "kudos" (arm-risc0-examples readme). the
elixir node runs juvix output as nock through a vm with a storage read opcode
(`lib/node/transaction/backends/backends.ex`, `nspec/.../notes/nockma.juvix.md`). on evm, forwarder
contracts let a transaction call other contracts; the adapter source says this "allows arbitrary
code execution through the protocol adapter but is constrained", and the generic-calls page says
there are "no restrictions on targets or function selectors" (`pa`, docs.anoma.net
anoma-generic-calls).

## provenance and what can be verified

proofs are verified on chain by risc zero verifier contracts; the adapter's stopped flag is
readable (`pa`). audit reports sit in the repositories: informal systems and nethermind on the
risc zero machine and the evm adapter (october to november 2025), informal systems on "anomapay
phase i" (december 2025), each with audited commit ids (arm-risc0 and pa-evm readmes, app sdk
`audits/`). reproducible guest builds are documented ("regenerate the elf and imageid"). the
research reports carry zenodo dois (art.anoma.net).

## trust assumptions

the adapter is `Ownable, Pausable` with an `emergencyStop()` for its owner; who holds that role on
each chain is not stated in the sources i read (`pa`). the off-chain transaction, proving and
indexer services of anomapay see plaintext and are not in the public organization listing. the
risc zero verifier router on each chain. the host chain's own consensus. for the elixir node,
consensus is an input: "all consensus is assumed to come in a form of an ordered list of binaries"
(`lib/node/transaction/mempool/mempool.ex`).

## open questions

the consensus design: the whitepaper names typhon, the spec deleted its consensus pages in 2024 and
names no algorithm in v0.2.0, the website says "novel", the roadmap puts native consensus in an
unreleased phase. whether the elixir node runs in any live deployment. the solver market (the node's
solver is "a strawman intent solver for testing purposes"). whether any deployment provides
function privacy. why the github repository shows pushes after april 2026 while its base branch
stays at 2026-04-09.
