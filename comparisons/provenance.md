# provenance

where each project's code, specification and history live, what an outsider can check, and what
each says about its own audits and builds. the repository numbers come from
[repository-history-tools](https://github.com/AlinaSchan/repository-history-tools), run on
2026-09-15 against the clones in [sources.md](../sources.md); rename detection off, merges counted
as commits without file changes, authors grouped by name after `.mailmap`.

## the repositories

| protocol | canonical host | license | spec and docs | first commit | commits | authors by name | source |
|---|---|---|---|---|---|---|---|
| darkfi | git.dark.fi (a gogs instance), copy on github | agpl 3.0 | the book in `doc/src` of the same repository; enhancement proposals in `doc/src/dep` | 2020-08-20 | 11,375 | 104 | `darkfi:README.md`, `LICENSE.md`, history tool |
| penumbra | github penumbra-zone/penumbra | mit or apache 2.0 | the protocol spec in `docs/protocol/src`; the guide moved to its own repository | 2021-03-25 | 6,851 | 115 | `penumbra:README.md`, `docs/guide/README.md`, history tool |
| anoma | github anoma/anoma (node), anoma/nspec (spec), anoma/arm-risc0, anoma/pa-evm | node mit, spec gpl 3.0, machine apache 2.0 | the spec is literate juvix, typechecked in ci | node 2023-07-17; spec 2024-03-07 | node 2,704; spec 334 | node 22; spec 13 | `anoma:LICENSE`, `nspec/../README.md`, github org listing, history tool |
| nym | github nymtech/nym (monorepo), nymtech/nyxd (chain) | libraries apache 2.0 or mit, binaries gpl 3.0, docs cc by-nc-sa 4.0 | docs in `documentation/docs` of the monorepo; the old docs repository is archived | 2019-11-25 | 11,209 | 119 | `nym:D/network/licensing.md`, `nym-docs/README.md`, history tool |
| namada | github namada-net/namada (moved from anoma/namada), anoma/namada-docs | apps gpl 3.0, libraries mit | docs and specs in a separate repository; genesis files in a third | 2021-02-02 | 17,544 | 96 | `namada:README.md`, `Cargo.toml`, history tool |

the counts say how much was committed and by how many names, nothing about how much work or how
many people. darkfi and nym have long histories with few merges (darkfi) or many (nym, 1,545);
namada's 3,039 merges and 17,544 commits over five years make it the largest of the five by commit
count; anoma's node history restarts in 2023 although its github repository was created in 2021,
and its spec has one author for 244 of 334 commits.

## what an outsider can verify

| protocol | verifiable from public material | source |
|---|---|---|
| darkfi | every proof against the circuits in the repository (no setup to trust); the chain state against the state root in each header; the fee total per height; the book against the code, since both are in one tree | `darkfi:doc/src/arch/consensus.md`, `doc/src/zkas/zkvm.md` |
| penumbra | proofs against verifying keys kept in git; chain state through ics-23 proofs and the state commitment tree anchors; signing test vectors; the ceremony's checks are specified and implemented, but the transcript is not in the repository | `penumbra:crates/crypto/proof-params/`, `SPEC/ibc.md`, `SPEC/transactions/signing.md`, `SPEC/setup/contributions.md` |
| anoma | on-chain proof verification through the risc zero verifier router; the adapter's stopped flag; reproducible guest builds ("regenerate the elf and imageid"); audited commit ids in the readmes | `pa`, arm-risc0 and pa-evm readmes, arm-risc0-examples readme |
| nym | the code; gateway probes anyone can run; a self-hostable node status api; from release 2026.17 a verifiable directory checked against a state proof, which the client core does not reference yet | `nym:D/operators/performance-and-testing/gateway-probe.mdx`, `docs/directory/README.md`, `CHANGELOG.md` |
| namada | masp parameters checked by size and blake2b on load; per-contribution ceremony files and a verify tool; chain state with `namadac`; the genesis repository | `namada:crates/shielded_token/src/validation.rs`, namada.net/trusted-setup, `ndocs:docs/users/query.mdx` |

## audits, as the projects themselves list them

| protocol | listed where | what is listed | source |
|---|---|---|---|
| darkfi | the book's start page | zksecurity, code audit, q1 2024 | `darkfi:doc/src/start-here.md` |
| penumbra | nowhere i looked: readme, spec, guide resources page, site home | none | grep over the repository, guide resources page |
| anoma | the readmes of arm-risc0 and pa-evm; the app sdk's `audits/` | informal systems and nethermind, october to november 2025, on the risc zero machine and the evm adapter; informal systems on anomapay phase i, december 2025 | arm-risc0 readme, pa-evm readme, anoma-app-sdk contents |
| nym | the website's trust center, not the docs | cure53 2024; oak security 2023 (twice); jp aumasson 2021; cryspen reviews "to be published" | nym.com/trust-center/independently-audited |
| namada | namada.net/security and one report in the repository | inference 2022 (in `audits/`); informal systems 2023 to 2025; least authority 2023; oak security 2023; antithesis ongoing | namada.net/security, `namada:audits/` |

"listed" is all this table says. i did not read the reports, and a listed audit says nothing about
what was in scope.

## reproducible builds

none of the five states that its release binaries are reproducible, as far as grep and the readmes
show: darkfi (not checked beyond the book), penumbra (cargo-dist on github ci, no statement), nym
(no mention), namada (no statement beyond genesis templates), anoma (guest programs documented as
reproducible, not the node). (`penumbra:dist-workspace.toml`, `namada:genesis/hardware/README.md`,
arm-risc0-examples readme)

## where the projects disagree with themselves

official sources that say different things about the same fact, found while reading:

- penumbra: the spec says fees are burned, the proto comment says a tip goes to the proposer; the
  spec's overview says sealed-bid swaps reveal only net flow, the dex chapter says swaps burn inputs
  publicly today; two governance descriptions. (`penumbra:SPEC/stake/validator-rewards.md`,
  `PROTO/core/component/fee/v1/fee.proto`, `SPEC/penumbra.md`, `SPEC/dex.md`)
- anoma: the whitepaper names typhon for consensus, the spec names nothing, the site says "novel";
  the spec's evm page says only sepolia is supported while nine mainnets are live.
  (whitepaper section 8.2, `nspec/arch/integrations/adapters/evm.md`, explorer.anoma.net)
- nym: two docs pages give different rewarded-set splits (60/80/100 and 120/50/70); three pages give
  different node counts. (`nym:D/operators/tokenomics/mixnet-rewards.mdx`, `D/operators/tokenomics.mdx`)
- namada: the spec lists the masp predicate as wasm, the code has it native; the launch page says
  phase 3, a later blog post says the launch is complete; docs and readme name different cometbft
  patch versions. (`ndocs:specs/base-ledger/execution.mdx`, `namada:crates/shielded_token/src/vp.rs`)
- darkfi: the overview page describes every wallet as a full node, the tooling page separates
  wallet and node, and the contributor guide marks the overview out of date; the transactions page's
  pseudo-code derives the signature key from the coin key, the circuit uses a separate secret.
  (`darkfi:doc/src/arch/overview.md`, `doc/src/dev/contrib/contrib.md`, `src/contract/money/proof/burn_v1.zk`)

none of these is a finding about the software. they are places where a reader has to pick the
source closest to the code, and say which one they picked.
