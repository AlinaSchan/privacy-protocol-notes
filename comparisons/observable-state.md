# observable state

what an outside observer can see, protocol by protocol. "outside observer" means someone with a
synced node or a public rpc and no keys. the five are not the same kind of object: darkfi,
penumbra and namada are ledgers; anoma's live deployments are contracts on other chains; nym's
chain is a registry and payment layer for a mixnet whose privacy lives off chain. rows that do not
apply say so.

citation keys: `darkfi:` darkrenaissance/darkfi at `a5514cb2`; `penumbra:` penumbra-zone/penumbra at
`36a31c17`, `SPEC` = `docs/protocol/src`, `PROTO` = `proto/penumbra/penumbra`; `nspec:` anoma/nspec
at `e850e7b2`, `docs/`; `pa:` anoma/pa-evm `contracts/src/ProtocolAdapter.sol`; `nym:` nymtech/nym at
`60f499eb`, `D` = `documentation/docs/pages`; `namada:` namada-net/namada at `024ff13d`; `ndocs:`
anoma/namada-docs at `ede86660`, `packages/docs/pages` and `packages/specs/pages`. full list in
[sources.md](../sources.md).

## transfers: amount, asset, counterparties

| protocol | public | hidden | source |
|---|---|---|---|
| darkfi | nullifier, merkle root, value and token commitments, spend hook, an encrypted user data field, signature key per input; coin commitment and encrypted note per output | value, token id, sender, recipient, which output is spent, memo | `darkfi:src/contract/money/src/model/mod.rs`, `darkfi:src/contract/money/proof/burn_v1.zk` |
| penumbra | nullifier, balance commitment, randomized verification key per spend; note commitment, ciphertext, ephemeral key per output; the action type ("our action is an enum") | amount, asset, sender, recipient; memo readable by sender and receivers only | `penumbra:SPEC/shielded_pool/action/spend.md`, `output.md`, `SPEC/concepts/transactions.md`, `SPEC/transactions/memo.md` |
| anoma (evm adapter) | commitments, nullifiers, application data blobs as events, tag and compliance unit counts, the outer logic key per resource, the erc-20 movements into and out of forwarders on the host chain | resource contents (label, value, quantity, nonce), nullifier keys, merkle paths; anomapay: sender, amount, receiver | `pa`, `nspec:arch/system/state/resource_machine/compliance_unit/compliance_proof.juvix.md`, docs.anoma.net anoma-pay-privacy-guarantees |
| nym | not a transfer ledger for users; nym token transfers on nyx are plain cosmos transfers | the mixnet hides who talks to whom, not token amounts | `nym:D/network/index.md`, `D/apis/cosmos-sdk-nyx.mdx` |
| namada | nullifiers, note commitments and anchors, the count of spend, output and convert descriptions, height and epoch; on shielding or unshielding, the asset and the amount, because the pool's balance is an ordinary balance key | for shielded-to-shielded: sender, receiver, asset, amount; note plaintexts and memos; payment addresses | `ndocs:specs/modules/masp/ledger-integration.mdx`, `namada:crates/shielded_token/src/storage_key.rs`, official blog post "understanding the masp and cc circuits" |

## the shape of a transaction

| protocol | what the shape reveals | source |
|---|---|---|
| darkfi | the tree of calls with contract ids; the function is in the payload; the number of inputs and outputs; proof sizes | `darkfi:doc/src/spec/concepts.md`, `doc/src/arch/sc/sc.md` |
| penumbra | the list of action types (spend, output, swap, delegate, vote, ...), fee, chain id, expiry, the number of detection clues | `penumbra:PROTO/core/transaction/v1/transaction.proto`, `SPEC/concepts/transactions.md` |
| anoma (evm adapter) | number of tags ("a multiple of two"), number of compliance units, which forwarder was called | `pa`, `nspec:arch/integrations/adapters/evm.md` |
| nym | packets are a fixed 2,413 bytes in mixnet mode; in dvpn mode "packets are not padded to a uniform size" | `nym:D/network/deep-dives/packet-anatomy.mdx`, `D/network/dvpn-mode/protocol.mdx` |
| namada | transaction type (raw, wrapper, protocol), the sections present (data, code, authorization, masp), batch and atomic flags, expiration, timestamp | `namada:crates/tx/src/section.rs`, `crates/tx/src/data/mod.rs` |

## fees

| protocol | public | source |
|---|---|---|
| darkfi | the fee amount of every transaction, native token only, and a per-block-height total | `darkfi:src/contract/money/src/entrypoint/fee_v1.rs` |
| penumbra | the fee amount and asset in the transaction body; the spec says fees are burned, the proto comment says a tip is paid to the proposer, the code path was not resolved in my reading | `penumbra:PROTO/core/component/fee/v1/fee.proto`, `SPEC/stake/validator-rewards.md` |
| anoma (evm adapter) | the host chain's gas, paid by whoever submits; for anomapay the transaction service submits "paying the necessary eth gas fees" | docs.anoma.net anoma-pay-architecture |
| nym | credentials are prepaid: a deposit on the ecash contract (deposit id, amount, claimed identity key) and gateways' redemption proposals; the spending of tickets is meant to be unlinkable to the deposit | `nym:common/cosmwasm-smart-contracts/ecash-contract/src/msg.rs`, `D/network/cryptography/zk-nym/unlinkability.mdx` |
| namada | the wrapper's fee token, price, gas limit and the fee payer's public key; a "disposable" gas payer funded by unshielding is the default for shielded transactions | `namada:crates/tx/src/data/wrapper.rs`, `ndocs:docs/users/fees.mdx` |

## who produced the block

| protocol | public | source |
|---|---|---|
| darkfi | a producer signature and the proof-of-work data on every block | `darkfi:doc/src/arch/consensus.md` |
| penumbra | cometbft validators with public identity and consensus keys, voting power, uptime | `penumbra:PROTO/core/component/stake/v1/stake.proto` |
| anoma | the host chain's block producers; the elixir node takes ordering as an input | `anoma:lib/node/transaction/mempool/mempool.ex` |
| nym | nyx validators, whitelisted, with voting power; the rewarded set of mix nodes and gateways per hour | `nym:D/operators/faq/nyx-faq.mdx`, `D/operators/tokenomics/mixnet-rewards.mdx` |
| namada | cometbft validators, consensus keys, bonds, slashes; validator metadata in genesis | `ndocs:specs/modules/proof-of-stake/objects-and-txs.mdx`, namada-mainnet-genesis readme |

## staking and delegation

| protocol | public | hidden | source |
|---|---|---|---|
| darkfi | proof of work, no staking | | `darkfi:doc/src/arch/overview.md` |
| penumbra | validator, epoch and amounts of every delegation and undelegation; validator pool sizes and exchange rates | the delegator's address; the amount of an unbonding claim | `penumbra:PROTO/core/component/stake/v1/stake.proto`, `SPEC/stake/delegation.md`, `SPEC/stake/action/undelegate_claim.md` |
| anoma | none on the adapter; xan governance is on ethereum | | docs.anoma.foundation tokenomics |
| nym | node bonds, delegations, operator cost and profit margin, rewards per epoch | | `nym:D/operators/nodes/nym-node/bonding.mdx` |
| namada | bonds, unbonds, redelegations, slashes, per validator and delegator | | `ndocs:specs/modules/proof-of-stake/objects-and-txs.mdx` |

## governance votes

| protocol | public | hidden | source |
|---|---|---|---|
| darkfi | that a dao vote transaction happened; what it reveals about its proposal is in `vote-main.zk`, which i did not trace | vote weight and voter, encrypted to the dao's votes key | `darkfi:doc/src/arch/dao.md`, `doc/src/start-here.md` |
| penumbra | validator votes, attributable; a delegator vote's power, validator, vote and proposal | the delegator's address; reusing a note across proposals links votes | `penumbra:SPEC/governance.md`, `SPEC/governance/action/delegator_vote.md` |
| anoma | xan governance on ethereum, outside the adapter | | anoma.net portal page |
| nym | nyx governance by validators; exit policy changes by operator vote off chain | | `nym:D/network/infrastructure/exit-services.mdx` |
| namada | every voter's vote under a storage key; tallies | | `ndocs:specs/modules/governance/storage.mdx` |

## trading

| protocol | public | hidden | source |
|---|---|---|---|
| darkfi | atomic swaps are built in the wallet; the on-chain form and whether it differs from a transfer is not clear from the public material | | `darkfi:doc/src/testnet/atomic-swap.md` |
| penumbra | every swap's trading pair and both input amounts ("publicly burn their input assets"), per-block batch totals, every liquidity position's prices, fees and reserves | the swapper's identity and claim address; the liquidity provider's identity; sealed-bid swaps are "a future upgrade" | `penumbra:SPEC/dex.md`, `SPEC/dex/swap.md`, `SPEC/dex/concentrated_liquidity.md` |
| anoma | anomapay swaps happen "on the venue", the host-chain dex | | docs.anoma.net anoma-pay-trust-model |
| nym | none | | |
| namada | no dex; the planned shielded swap on osmosis unshields to a transparent osmosis account, swaps "openly" and returns | | official blog post "shielded swaps on osmosis" |

## the size of the anonymity set

| protocol | what can be counted | source |
|---|---|---|
| darkfi | coins ever created and nullifiers ever published, so the number of unspent coins; no totals of value | `darkfi:src/contract/money/src/lib.rs` |
| penumbra | inbound ibc transfers are public (sender, amount, denom), so per-asset flows into and out of the shielded pool are derivable; note commitments and nullifiers | `penumbra:SPEC/ibc.md`, guide privacy page |
| anoma (evm adapter) | commitments and nullifiers per adapter; erc-20 balances of the forwarders on the host chain | `pa` |
| nym | not applicable to a ledger; the mixnet's anonymity set is the traffic of all clients in the same hour's rewarded set | `nym:D/network/mixnet-mode.mdx` |
| namada | the pool's balance per asset, as a transparent storage key; note commitments and nullifiers | `namada:crates/shielded_token/src/storage_key.rs`, `ndocs:specs/modules/masp/ledger-integration.mdx` |

## derivable with effort

- darkfi: activity per contract function over time; producer keys' share of blocks; transaction
  shape as a fingerprint. (`darkfi:doc/src/arch/sc/sc.md`, `doc/src/arch/consensus.md`)
- penumbra: clearing prices from batch outputs; amount correlations across ibc deposits,
  unbondings and position withdrawals; a client's blocks of interest from which full transactions
  it fetches; address linkage by a detector holding several detection keys.
  (`penumbra:SPEC/dex/swap.md`, `crates/view/src/worker.rs`, `SPEC/addresses_keys/addresses.md`)
- anoma: creation-time bounds from the accumulator root a proof used; consumption time from
  nullifier publication; the link between a shielded transfer and the transaction service's
  submitting address. (`nspec:.../commitment_accumulator.juvix.md`,
  `.../roles_and_requirements.juvix.md`, docs.anoma.net anoma-pay-privacy-guarantees)
- nym: node uptime history from rewards, stake concentration, gateway usage from redemption counts
  once redemption runs. (`nym:D/operators/tokenomics/mixnet-rewards.mdx`, ecash contract)
- namada: "if one account shields then unshields an asset, we can trace the btc back to their
  origin"; the fee payer when set explicitly; linkage when masp funds pay for transparent
  transactions. (`ndocs:docs/users/shielded-accounts.mdx`, `docs/users/fees.mdx`)

## where the comparison breaks

nym's chain is public on purpose; its question is the network, see
[network-metadata.md](network-metadata.md). anoma's privacy today is the privacy of a contract
deployment on a transparent chain plus off-chain services that see plaintext; the spec's shielded
machine is a design with a "tbd" marker. penumbra hides transfers and reveals trades and stakes;
namada hides transfers and reveals the pool's totals; darkfi hides values and reveals the shape.
none of the five is a superset of another.
