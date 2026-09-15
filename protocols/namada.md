# namada

a proof-of-stake chain whose shielded pool holds many assets in one anonymity set, with ibc
transfers in and out of it. written against the namada repository at commit `024ff13d`
(2026-09-10) and the namada-docs repository at `ede86660` (2025-12-02). citation keys: `docs` is
`packages/docs/pages` in namada-docs (docs.namada.net), `specs` is `packages/specs/pages`
(specs.namada.net), `genesis` is the mainnet genesis repository namada-net/namada-mainnet-genesis.

## in its own words

"proof-of-stake l1 for multichain asset-agnostic data protection" (`README.md`); "shielded asset
hub rewarding you to protect the multichain" (namada.net, read 2026-09-15). the readme also warns
"this codebase is still experimental".

## status

mainnet chain `namada.5f5de2dd1b88cba30586420`, genesis 2024-12-03 15:00 utc, 205 genesis
validators (`docs/networks/mainnets.mdx`, `genesis/MAINNET-GUIDE.md`, `genesis/README.md`). the
launch ran in five phases switched on by governance; a blog post of june 2025 says the launch is
complete and nam transfers are enabled, while the mainnet-launch page still says "phase 3"
(namada.net/mainnet-launch, official blog post). github.com/anoma/namada redirects to
namada-net/namada; the workspace author is heliax ag (`Cargo.toml`). cometbft 0.37 through a shim
that runs transactions in end-block (`crates/node/src/shims/abcipp_shim.rs`). apps gpl 3.0, libraries
mit.

## public state

- transparent balances, and the shielded pool's own balance per asset, which is an ordinary
  balance key of the masp address: "the asset balance of the shielded pool itself is transparent"
  (`specs/modules/masp/ledger-integration.mdx`, `crates/shielded_token/src/storage_key.rs`)
- validator sets, consensus keys, bonds, unbonds, slashes; governance proposals with every voter's
  vote under a storage key; public goods funding stewards (`specs/modules/proof-of-stake/`,
  `specs/modules/governance/storage.mdx`)
- per transaction: the wrapper's fee token, price, gas limit and fee payer public key; timestamp,
  expiration, chain id (`crates/tx/src/data/wrapper.rs`, `crates/tx/src/section.rs`)
- per shielded transaction: nullifiers, note commitments and anchors, the number of spend, output
  and convert descriptions, the per-asset value balance whenever value enters or leaves the pool,
  the height and epoch (`ledger-integration.mdx`, `storage_key.rs`)
- the conversion table that drives shielded rewards (`storage_key.rs`)

## private state

for shielded-to-shielded transfers: sender, receiver, asset and amount; note plaintexts and memos
travel encrypted; payment addresses are "not visible on-chain"; the pairing of a nullifier with its
note (`ledger-integration.mdx`, `docs/users/addresses.mdx`, official blog post on the masp).
shielding and unshielding reveal the asset and the amount, because the pool's balance moves
(`masp` readme). the fee payer can be a "disposable" address funded by unshielding, the default
for shielded transactions (`docs/users/fees.mdx`).

## identity assumptions

an implicit address is the hash of a public key and exists as soon as the key does; an established
account is created on chain and is a k-of-n multisig, 1-of-1 by default, with the `vp_user`
validity predicate (`docs/users/transparent-accounts/`, `crates/core/src/address.rs`). shielded
keys: spending key, viewing key, payment address, one viewing key for many addresses, zip-32
derivation; a key can carry a "birthday" height to bound scanning (`docs/users/shielded-accounts/
masp-keys.mdx`). viewing keys are the disclosure mechanism, for accounting or compliance (official
blog post). validators are accounts with a consensus key and "may not also be a delegator"
(`specs/modules/governance.mdx`).

## metadata exposure and network-level privacy

the docs advise tor or a vpn and say bridging in or out "can still expose personal information",
worst for nfts and thin tokens (`docs/introduction/privacy.mdx`). a wallet syncing the shielded pool
needs a local copy of all notes; the sdk requests masp events and blocks by height range from an
rpc, or note ranges from a masp indexer, and the web wallet depends on an rpc, an indexer and a masp
indexer "operated by the same provider" (`docs/users/shielded-accounts/shielded-sync.mdx`,
`crates/sdk/src/masp/utilities.rs`, `docs/users/namadillo/overview.mdx`). shielded ibc: the
counterparty transfer names the masp address as receiver and carries the shielded transaction in
the memo, so the sending chain records sender, the masp address, denom, amount and memo
(`docs/users/ibc/shielded-ibc.mdx`, `crates/ibc/src/msg.rs`). no threat-model page was found in
the docs tree.

## programmability

transactions and validity predicates are wasm, but "only a selected subset" runs: allowlists of
hashes, 2 predicates and 22 transactions at mainnet genesis, changed by governance
(`specs/base-ledger/execution.mdx`, `genesis/parameters.toml`). proof of stake, ibc, governance,
the multitoken and the masp are native predicates in the node (`crates/shielded_token/src/vp.rs`).
the spec lists the masp predicate as wasm; the code has it native.

## provenance and what can be verified

the masp extends zcash sapling with an asset type per note; groth16 over bls12-381 for spend,
output and convert; parameters checked by size and blake2b hash on load
(`specs/modules/masp.mdx`, `crates/shielded_token/src/validation.rs`). the trusted setup ran from
2022-11-19 to 2022-12-21 with 2,510 contributions and a public random beacon from bitcoin, ethereum
and zcash block hashes; per-contribution files and a verify tool are published (official blog post,
namada.net/trusted-setup). audits listed on namada.net/security: inference 2022 (the report is in
`audits/`), informal systems 2023 to 2025, least authority 2023, oak security 2023. the masp readme
says its crates "have not been fully-reviewed". no reproducible-build statement found.

## trust assumptions

one honest participant in the setup; the validator set; the rpc, indexer and masp indexer a wallet
talks to; governance, which controls the allowlists, reward parameters and conversions; the
counterparty chain and relayers for ibc.

## open questions

block time (docs say four seconds, the config template says a one second commit timeout); live
parameter values after governance changes; which launch phase is current; whether the ethereum
bridge, present in code behind a feature flag and disabled at genesis, will be deployed; whether
"pinned" shielded transactions from the spec exist in code.
