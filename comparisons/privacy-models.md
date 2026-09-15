# privacy models

what each system is designed to hide, from whom, and with which primitives. citation keys as in
[observable-state.md](observable-state.md); full list in [sources.md](../sources.md).

## what is hidden, and from whom

| protocol | hidden from a ledger observer | hidden from a network observer | hidden from the counterparty or operator | source |
|---|---|---|---|---|
| darkfi | values, tokens, keys, spend links, memos; dao membership and votes | only what the chosen transport (tor, i2p, nym) hides; the p2p layer adds nothing | a recipient learns the note; a node operator sees what it relays | `darkfi:doc/src/arch/tx_lifetime.md`, `doc/src/start-here.md` |
| penumbra | amount, asset, sender, recipient; delegator and voter addresses; lp identity | nothing by design: cometbft gossip; fmd threat model assumes tor and https | the view server holds the full viewing key; a detector can link addresses | `penumbra:SPEC/shielded_pool/action/spend.md`, `SPEC/crypto/fmd/threat_model.md`, guide pclientd page |
| anoma (anomapay) | sender, amount, receiver on the host chain | nothing stated; the intent goes to a transaction service | the transaction and proving services "receive your witness data" | docs.anoma.net anoma-pay-privacy-guarantees, anoma-pay-trust-model |
| nym | not the subject | who talks to whom, when, how often, how much; client ip from mix nodes, exit and destination | the entry gateway sees the client ip; the exit sees the destination and unencrypted content; the destination sees the exit | `nym:D/network/index.md`, `D/network/infrastructure/exit-services.mdx` |
| namada | sender, receiver, asset, amount of shielded transfers; payment addresses | nothing by design; docs advise tor or a vpn | the rpc and masp indexer see which heights a wallet fetches; a viewing key holder sees everything | `ndocs:docs/introduction/privacy.mdx`, `docs/users/shielded-accounts/shielded-sync.mdx` |

## primitives

| protocol | proof system | curve or group | commitments | in-circuit hash | note encryption | trusted setup | source |
|---|---|---|---|---|---|---|---|
| darkfi | halo2, one circuit per zkas program, no trusted setup | pallas and vesta | pedersen for values (homomorphic); poseidon for coins, token ids and bullas | poseidon (x^5, 8 full, 56 partial rounds) | chacha20-poly1305 with a blake2b-derived key; an elgamal-style scheme with poseidon where a circuit must verify the ciphertext | none | `darkfi:doc/src/zkas/zkvm.md`, `doc/src/spec/crypto-schemes.md`, `src/contract/money/proof/burn_v1.zk` |
| penumbra | groth16, seven circuits (spend, output, swap, swapclaim, convert, delegator vote, nullifier derivation) | bls12-377 with the decaf377 group | pedersen balance commitments with per-asset generators; poseidon note commitments | poseidon; blake2b-512 outside circuits | chacha20-poly1305 per action, keys wrapped for sender recovery | yes: a two-phase ceremony run with `summonerd`; parameters committed 2024-06-25; participants and transcript not in the repository | `penumbra:SPEC/crypto.md`, `SPEC/crypto/decaf377.md`, `SPEC/addresses_keys/transaction_crypto.md`, `SPEC/setup.md`, git `595966fc3` |
| anoma (arm-risc0) | risc zero zkvm: stark proofs, groth16 wrapping for on-chain verification | secp256k1 for the delta proof (ecdsa) | commitment and nullifier hashes; sha-256 over abi-encoded data in the evm adapter | sha-256 in the evm adapter | aes-256-gcm with a k256 shared key (gadget) | stark: none; the groth16 wrapper's setup is not covered by the sources i read | `nspec:arch/integrations/adapters/evm.md`, arm-risc0 readme and `arm/src/delta_proof.rs`, `arm_gadgets/src/encryption.rs` |
| nym | no zk on the traffic path; credentials use blinded threshold signatures with pedersen commitments and nizk proofs | curve25519 and x25519 in packets and wireguard; bls12-381 named in an audit finding on the coconut library | not a ledger of commitments | not applicable | sphinx per-hop ecdh, hkdf, hmac, lioness; wireguard chacha20-poly1305 | none for zk; a dkg among the quorum for credential keys | `nym:D/network/cryptography/sphinx.md`, `D/network/cryptography/zk-nym.mdx`, `D/operators/nodes/validator-setup/nym-api.mdx` |
| namada | groth16 (sapling extended with an asset type), spend, output and convert circuits | bls12-381 | sapling-style value commitments per asset type; note commitments | sapling's hashes; blake2s for asset types | sapling-style note ciphertexts (612 bytes) | yes: phase 2 mpc over zcash's powers of tau, 2022-11-19 to 2022-12-21, 2,510 contributions, public random beacon | `ndocs:specs/modules/masp.mdx`, `specs/modules/masp/ledger-integration.mdx`, `specs/modules/masp/trusted-setup.mdx`, official blog post on the ceremony |

## three ways to get programmable privacy

the ledgers differ most in where the circuits come from.

- darkfi compiles a circuit per contract from zkas source and runs it on one halo2 virtual
  machine; adding an application means writing new circuits, and the proving system needs no
  ceremony. (`darkfi:doc/src/zkas/zkvm.md`)
- penumbra and namada fix the set of circuits: penumbra's seven actions and namada's three sapling
  circuits. new behaviour arrives as protocol upgrades and governance parameters, not as user
  programs, and each circuit needed its own setup. (`penumbra:SPEC/crypto/proofs.md`,
  `ndocs:specs/base-ledger/execution.mdx`)
- anoma puts the application logic inside the proof as a risc zero guest program; the spec's
  function privacy needs a recursive proof of the logic proof, which the deployed evm circuit does
  not provide. (`nspec:.../action/index.juvix.md`, `nspec:arch/integrations/adapters/evm.md`)

nym is on a different axis: it does not make a ledger private, it makes the path to the ledger
private, and its docs say so at the start ("expose[s] ip addresses, timing, and other network
metadata" even with zero-knowledge proofs on chain). (whitepaper p.1)

## the trusted setup as a trust assumption

| protocol | assumption | what is published | source |
|---|---|---|---|
| darkfi | none | | `darkfi:doc/src/zkas/zkvm.md` |
| penumbra | at least one honest participant per phase | verifying keys in git, proving keys hash-pinned; contribution checks specified; no participant count, dates or transcript in the repository | `penumbra:crates/crypto/proof-params/src/lib.rs`, `SPEC/setup/contributions.md` |
| anoma (risc zero) | none for the stark; the groth16 wrapper is risc zero's | not covered here | arm-risc0 readme |
| nym | not applicable | | |
| namada | "1 out of n" of 2,510 contributions | per-contribution hash, signature and response files, a verify tool, a beacon derived from bitcoin, ethereum and zcash block hashes | official blog posts on the ceremony, namada.net/trusted-setup |

## selective disclosure

| protocol | mechanism | source |
|---|---|---|
| darkfi | none documented; "money::transfer() contract viewing keys" is an open task | `darkfi:doc/src/dev/contrib/contrib.md` |
| penumbra | full, incoming and outgoing viewing keys; per-transaction "transaction perspective"; detection keys for fuzzy message detection can be outsourced | `penumbra:SPEC/addresses_keys/viewing_keys.md`, guide disclosure page |
| anoma (anomapay) | an encryption key per user; payloads readable by the owner; the services already hold the witness | docs.anoma.net key-management, anoma-pay-trust-model |
| nym | not applicable; credentials disclose an expiration date and ticket type, nothing else | `nym:common/nym_offline_compact_ecash/src/constants.rs` |
| namada | viewing keys, shared with third parties "for accounting or compliance" | `ndocs:docs/users/addresses.mdx`, official blog post on data protection |

## the same words, different things

"shielded" in namada means inside the masp; in anoma's spec it means data and function privacy
together, and by that definition namada's pool and the deployed anoma adapter are both transparent
about which logic ran. "anonymous" in darkfi covers values, keys and dao membership but not the
transaction shape. "private" in nym's pages is a statement about network observers and is graded
"partial" against a global one. reading each project with its own definitions is the only way the
comparison holds. (`nspec:arch/system/state/resource_machine/index.juvix.md`,
`nym:documentation/docs/lib/privacy-model/examples/generic.ts`)
