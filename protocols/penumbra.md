# penumbra

a proof-of-stake chain with a shielded pool, staking, governance and a decentralized exchange,
running its own rust application under cometbft. written against the penumbra repository at commit
`36a31c17` (2026-01-24, the last commit on the github main branch when i cloned it on 2026-09-15).
citation keys: `SPEC` is `docs/protocol/src` in the repository, rendered at protocol.penumbra.zone;
`PROTO` is `proto/penumbra/penumbra`; `guide` is guide.penumbra.zone.

## in its own words

"a fully shielded zone for the Cosmos ecosystem" (`README.md`); "a fully private proof-of-stake
network and decentralized exchange" (`SPEC/penumbra.md`).

## status

the compatibility table labels app versions 7 to 12 "mainnet" (`COMPATIBILITY.md`); tags `v0.79.0`
2024-07-05, `v1.0.0` 2025-01-21, `v2.1.1` 2026-01-24. the chain id is `penumbra-1` (guide). the
launch date itself is not stated in the repository or the guide pages i read. penumbra.zone
redirects to penumbra.exchange (curl, 2026-09-15). not a cosmos sdk chain: `pd` is a tower-abci
application in rust, cometbft runs as a separate process (`crates/bin/pd/src/main.rs`, guide).
license mit or apache 2.0, copyright penumbra labs.

## public state

- action types: a transaction's actions are an enum, and the spec says the type is not concealed
  (`SPEC/concepts/transactions.md`); fee, chain id, expiry and clue count sit in the transaction
  body (`PROTO/core/transaction/v1/transaction.proto`)
- per spend: nullifier, balance commitment, randomized verification key; per output: note
  commitment, ciphertext, ephemeral key, balance commitment (`SPEC/shielded_pool/action/spend.md`,
  `output.md`)
- swaps today "publicly burn" inputs: the trading pair and both input amounts are in the clear,
  and per-block batch totals are published; sealed-bid swaps are "a future upgrade"
  (`SPEC/dex.md`, `SPEC/dex/swap.md`, `PROTO/core/component/dex/v1/dex.proto`)
- liquidity positions with prices, fees and reserves; position opens, closes, withdrawals
  (`SPEC/dex/concentrated_liquidity.md`)
- delegations and undelegations reveal the validator and the amounts; private delegation is "for
  a future network upgrade" (`PROTO/core/component/stake/v1/stake.proto`,
  `SPEC/stake/delegation.md`)
- validator definitions, states, voting power, exchange rates, uptime, penalties
  (`stake.proto`)
- governance: validator votes are public and attributable; a delegator vote hides the address but
  reveals voting power, validator, vote and proposal (`SPEC/governance.md`)
- ibc withdrawals reveal amount, denom and destination
  (`PROTO/core/component/ibc/v1/ibc.proto`, guide privacy page)

## private state

transfer amount, asset, sender and recipient; the swapper's identity and claim address; the
delegator's address; the liquidity provider's identity; the memo; the amount of an unbonding claim
(`SPEC/shielded_pool/action/spend.md`, `SPEC/dex/action/swap.md`, guide privacy page,
`SPEC/stake/action/undelegate_claim.md`).

## identity assumptions

"only validators have any kind of long-term identity" (`SPEC/penumbra.md`). keys derive from a
bip44 path (coin type 6532) into a spend key, a full viewing key, an incoming and an outgoing
viewing key; each account has 2^128 unlinkable diversified addresses scanned with one incoming
viewing key (`SPEC/addresses_keys/`). every address carries a clue key for fuzzy message
detection, and a detection key can be handed to a third party to scan on the owner's behalf; the
spec itself notes that a detector holding several detection keys can link addresses
(`SPEC/addresses_keys/addresses.md`, `SPEC/crypto/fmd/`). disclosure is per transaction through a
"transaction perspective" (guide disclosure page).

## metadata exposure and network-level privacy

consensus gossip is cometbft's, on 26656/tcp (guide). the view server `pclientd` holds the full
viewing key and "does not support any kind of transport security" (`crates/bin/pclientd/src/lib.rs`).
a client scans compact blocks and fetches full transactions only for blocks where it detected
something, so a node can see which heights a client cares about ("chaff downloads" are a todo,
`crates/view/src/worker.rs`). the fmd threat model assumes tor and https between user and detector
(`SPEC/crypto/fmd/threat_model.md`).

## programmability

none in the smart contract sense. the action set is fixed protocol logic in component crates, and
the spec says programmability "isn't a near-term objective" (`SPEC/crypto/proofs.md`,
`crates/core/component/`). parameters change through governance, some upgrades need a chain halt
(`SPEC/governance.md`).

## provenance and what can be verified

groth16 over bls12-377 with the decaf377 group, one circuit per action type, seven proving keys
hash-pinned in the repository (`SPEC/crypto.md`, `crates/crypto/proof-params/`). the trusted setup
was a two-phase ceremony run with the project's `summonerd`; the final parameters were committed on
2024-06-25. the participant count, dates and transcript are not in the repository. no third-party
audit is linked from the readme, the spec, the guide's resources page or the website's home. no
reproducible-build statement (`dist-workspace.toml`, grep).

## trust assumptions

the setup ceremony (one honest participant among those who took part); the validator set for
ordering; the operator of whatever rpc and view server a wallet uses; the detector, if detection is
outsourced. the spec says fees "are burned", the proto comment says a tip is "paid to the
proposer", and the code's fee path was not resolved in my reading (`SPEC/stake/validator-rewards.md`,
`PROTO/core/component/fee/v1/fee.proto`).

## open questions

launch date and genesis; the ceremony's participants; live mainnet parameters (only code defaults
seen, epoch 17,280 blocks); which of two governance descriptions in the spec is current; whether
main has moved since january 2026 on another host.
