# identity

who has to be someone in each system, how keys and addresses are built, and who can be asked to
prove something. citation keys as in [observable-state.md](observable-state.md); full list in
[sources.md](../sources.md).

## accounts and addresses

| protocol | accounts | how an address is built | addresses per key | source |
|---|---|---|---|---|
| darkfi | none; a coin commits to its owner's public key | public key = secret times a fixed generator on pallas; addresses are keys | one key, but no address appears on chain: coins carry commitments, notes carry ciphertexts | `darkfi:doc/src/spec/crypto-schemes.md`, `src/contract/money/proof/mint_v1.zk` |
| penumbra | none; "only validators have any kind of long-term identity" | bip44 path with coin type 6532 to a spend key, then full, incoming and outgoing viewing keys; diversified addresses with a clue key each, bech32m `penumbra` | 2^128 unlinkable addresses per account, all scanned with one incoming viewing key | `penumbra:SPEC/penumbra.md`, `SPEC/addresses_keys/spend_key.md`, `SPEC/addresses_keys/addresses.md` |
| anoma | resources have owners through nullifier key commitments; identities are "defined entirely on the basis of whether or not they know some secret information" | anomapay derives an authority, a nullifier, an encryption and a discovery key from one seed or from an evm wallet signature | not stated | `nspec:arch/system/identity/identity.juvix.md`, docs.anoma.net key-management |
| nym | node accounts on nyx (bech32 `n`); clients have no chain account | node: ed25519 identity plus rotating x25519 sphinx keys; client: ed25519 identity plus x25519 key, address `identity.encryption@gateway` | a client address embeds its entry gateway and is stable unless the client rotates keys | `nym:nym-node/src/config/persistence.rs`, `D/network/reference/addressing.md` |
| namada | implicit accounts (hash of a public key), established accounts (created on chain, k-of-n multisig), internal accounts | transparent `tnam`, public key `tpknam`; shielded: spending key `zsknam`, viewing key `zvknam`, payment address `znam`, zip-32 derivation | "very large number" of payment addresses per viewing key; payment addresses "not visible on-chain" | `ndocs:docs/users/addresses.mdx`, `namada:crates/core/src/address.rs`, `crates/core/src/string_encoding.rs` |

## who is identified by design

| protocol | identified | not identified | source |
|---|---|---|---|
| darkfi | block producers, by signing key; nodes, by transport address and version message; a mint authority when it mints or freezes a token | senders, recipients, dao members, voters | `darkfi:doc/src/arch/consensus.md`, `src/net/message.rs`, `src/contract/money/proof/` |
| penumbra | validators (identity key, consensus key, governance key, name, website); delegations by validator and amount; validator votes | delegators, voters, liquidity providers, swappers, senders, recipients | `penumbra:PROTO/core/component/stake/v1/stake.proto`, `SPEC/governance.md` |
| anoma | the adapter owner and its emergency stop caller; the transaction service's submitting address; anomapay users to the operator, who runs kyc and aml checks | sender and receiver of a shielded transfer on chain | `pa`, docs.anoma.net anoma-pay-trust-model |
| nym | node operators (bond, owner address, cost, margin); validators (whitelisted); credential buyers at deposit time, by nyx address | which client sent which packet; which ticket came from which deposit | `nym:D/operators/nodes/nym-node/bonding.mdx`, `D/network/cryptography/zk-nym/zk-nym-overview.mdx` |
| namada | validators and delegators; voters; fee payers by public key unless a disposable payer is used; established accounts and their signers | shielded senders and receivers | `ndocs:specs/modules/proof-of-stake/objects-and-txs.mdx`, `specs/modules/governance/storage.mdx`, `docs/users/fees.mdx` |

## the key that signs is not the key that owns

two of the ledgers separate the spend authorization key from the coin's key, so that the public key
that appears with a spend cannot be matched to the key in the commitment.

- darkfi's burn circuit takes `signature_secret` as a separate witness from `coin_secret`; the
  chain sees a key derived from the former. (`darkfi:src/contract/money/proof/burn_v1.zk`)
- penumbra publishes a randomized verification key `rk` per spend, from a reddsa variant with
  randomizable keys. (`penumbra:SPEC/crypto/decaf377-rdsa.md`, `SPEC/shielded_pool/action/spend.md`)
- namada follows sapling, where the spend authorization key is re-randomized; the namada sources
  i read describe the masp as a sapling extension and do not restate this step, so i leave it at
  that. (`ndocs:specs/modules/masp.mdx`)
- anoma's delta proof is an ecdsa signature over secp256k1 in the risc zero machine; what key
  signs it and whether it is randomized per transaction is not clear from the public material i
  read. (`arm/src/delta_proof.rs`)

## disclosure and delegation of viewing

| protocol | can hand out | to whom it is meant for | what the holder can do | source |
|---|---|---|---|---|
| darkfi | nothing documented for money; the dao has six keys (view notes, propose, view proposals, view votes, execute, early execute) that a founder can assign | dao roles | read or act on one dao function | `darkfi:doc/src/arch/dao.md`, `doc/src/dev/contrib/contrib.md` |
| penumbra | full viewing key, incoming viewing key, outgoing viewing key, a detection key per address, a transaction perspective | auditors, view servers, detectors | scan, decrypt, or detect probable matches; a detector with several detection keys can link addresses | `penumbra:SPEC/addresses_keys/viewing_keys.md`, `SPEC/addresses_keys/addresses.md`, guide disclosure page |
| anoma | the encryption key reads payloads; the discovery key is registered with the indexer | the user's own wallet and the indexer | read one's own resources; the operator already holds the witness | docs.anoma.net key-management, anoma-pay-trust-model |
| nym | nothing; a credential discloses its expiration date and ticket type only | | | `nym:common/nym_offline_compact_ecash/src/constants.rs` |
| namada | viewing keys, with an optional birthday height to bound scanning | "third parties, for accounting or compliance"; indexers | see incoming and outgoing balances, cannot spend | `ndocs:docs/users/addresses.mdx`, `docs/users/shielded-accounts/masp-keys.mdx`, official blog post on data protection |

## node and validator identity

- darkfi nodes carry an optional `node_id` in the version message, commented "compromises privacy
  when set", and advertise external addresses only if configured. (`darkfi:src/net/message.rs`)
- penumbra and namada validators are cometbft validators with public consensus keys; namada adds
  metadata such as an email address in the genesis files. (`penumbra:stake.proto`,
  namada-mainnet-genesis readme)
- nym nodes bond with an ed25519 identity from a nyx wallet and keep a second nyx account for daily
  config pushes and key rotation, which appear on chain about once a day.
  (`nym:D/operators/nodes/nym-node/bonding.mdx`)
- anoma's spec gives every node an id that is a public key and node adverts that are signed; the
  live deployments are contracts, and the question of node identity does not arise for them.
  (`nspec:arch/node/types/identities.juvix.md`, `nspec:arch/node/engines/net_registry_messages.juvix.md`)
