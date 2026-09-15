# sources

primary sources only. repositories were cloned from github on 2026-09-15; the commit hashes below
are what the pages were written against. websites and blog posts were read on the same day.

## darkfi

- repository: https://git.dark.fi/darkrenaissance/darkfi (the readme's clone url; a gogs instance),
  copy at https://github.com/darkrenaissance/darkfi, commit `a5514cb2cecdc791ed8de68440455433beeebe7a`
  (2026-09-10)
- the book, built from `doc/src`: https://dark.fi/book/
- pages and files cited: `README.md`, `LICENSE.md`, `doc/src/start-here.md`,
  `doc/src/arch/{overview,consensus,tx_lifetime,dao,tooling}.md`, `doc/src/arch/sc/sc.md`,
  `doc/src/arch/net/p2p-network.md`, `doc/src/spec/{concepts,crypto-schemes}.md`,
  `doc/src/zkas/zkvm.md`, `doc/src/dep/0007.md`, `doc/src/testnet/{node,atomic-swap}.md`,
  `doc/src/dev/contrib/contrib.md`, `src/contract/money/proof/{burn_v1,mint_v1}.zk`,
  `src/contract/money/src/{lib.rs,model/mod.rs,entrypoint/fee_v1.rs}`, `src/net/message.rs`,
  `src/net/transport/`, `bin/darkfid/src/rpc/`
- website: https://dark.fi
- longer notes: https://github.com/alinaschanz/darkfi-protocol-notes

## penumbra

- repository: https://github.com/penumbra-zone/penumbra, commit
  `36a31c17974c23a7e84cc02c64f0062ae57e79b1` (2026-01-24, the head of the main branch on the clone date)
- protocol spec, built from `docs/protocol/src`: https://protocol.penumbra.zone
- guide: https://guide.penumbra.zone (source https://github.com/penumbra-zone/guide); pages read:
  overview/privacy, overview/disclosure, network/node/pclientd, network/node/pd/join-network,
  network/relayers, dev/parameter_setup, resources
- files cited: `README.md`, `COMPATIBILITY.md`, `docs/protocol/src/**` as named in the pages,
  `proto/penumbra/penumbra/core/**/*.proto`, `crates/bin/pclientd/src/lib.rs`,
  `crates/view/src/worker.rs`, `crates/crypto/proof-params/src/lib.rs`, `dist-workspace.toml`,
  commit `595966fc3` (ceremony parameters), git tags
- website: https://penumbra.zone, which redirected to https://penumbra.exchange on the reading date

## anoma

- specification: https://github.com/anoma/nspec, commit `e850e7b29d113adbf189bf805837595acd6d0ac2`
  (2025-06-03, tag v0.2.0), rendered at https://specs.anoma.net
- node: https://github.com/anoma/anoma, commit `5bcbf8487a331a20748299ed4fd4d62027f0ffa8`
  (2026-04-09, branch `base`)
- read on github without cloning: https://github.com/anoma/arm-risc0 (readme, `arm/src/resource.rs`,
  `arm/src/delta_proof.rs`, `arm/src/compliance.rs`, `arm_gadgets/src/encryption.rs`),
  https://github.com/anoma/pa-evm (readme, `contracts/src/ProtocolAdapter.sol`),
  https://github.com/anoma/arm-risc0-examples, https://github.com/anoma/anoma-app-sdk,
  https://github.com/anoma/anoma-protobuf (`priv/protobuf/anoma.proto`), the organization's
  repository list through the github api
- whitepaper: "anoma: a unified architecture for full-stack decentralised applications", goes, sun
  yin, brink, https://github.com/anoma/whitepaper (text dated 2022-08-21, pdf built 2025-08-07)
- research reports index: https://art.anoma.net
- documentation pages: https://docs.anoma.net (anoma-pay-privacy-guarantees, anoma-pay-trust-model,
  anoma-pay-architecture, key-management, anoma-generic-calls, sdk-overview, anoma-explorer)
- website and explorer: https://anoma.net (home, tech-stack, roadmap, about), https://explorer.anoma.net
  (networks, contracts), https://docs.anoma.foundation/tokenomics
- official blog posts, by slug under https://anoma.net/blog/: helloworld-anomas-first-devnet-is-live
  (2025-01-16), the-anoma-testnet-is-live (2025-07-15), xan-is-live (2025-09-29),
  anoma-is-now-live-on-ethereum-mainnet (2025-11-18), anomapay-v1-is-live (2026-06-02),
  the-anoma-roadmap (2026-07-08)

## nym

- repository: https://github.com/nymtech/nym, commit `60f499eb28e087e6902d8aa61d55060337648729`
  (2026-09-15); docs source `documentation/docs/pages`, rendered at https://nym.com/docs; the old
  docs repository https://github.com/nymtech/docs is archived (clone at `63c72aa7`, 2023-04-14)
- files cited: `documentation/docs/pages/network/**`, `.../operators/**`, `.../developers/**`,
  `.../apis/**`, `documentation/docs/lib/privacy-model/{threat-model.ts,examples/generic.ts}`,
  `common/nymsphinx/params/src/packet_sizes.rs`, `common/client-core/config-types/src/lib.rs`,
  `common/topology/src/lib.rs`, `common/gateway-requests/src/registration/handshake/messages.rs`,
  `common/nym_offline_compact_ecash/src/constants.rs`,
  `common/cosmwasm-smart-contracts/ecash-contract/src/msg.rs`, `common/network-defaults/src/mainnet.rs`,
  `nym-node/src/config/persistence.rs`, `docs/directory/README.md`, `docs/LP_PROTOCOL.md`,
  `CHANGELOG.md`
- whitepaper: "the nym network: the next generation of privacy infrastructure", diaz, halpin,
  kiayias, version 1.0, 2021-02-26, https://nym.com/nym-whitepaper.pdf
- papers the docs cite: loopix (piotrowska et al., usenix security 2017), sphinx (danezis and
  goldberg, ieee s&p 2009), coconut (sonnino et al., ndss 2019), the offline ecash paper (arxiv
  2303.08221)
- website: https://nym.com (home, trust-center, trust-center/cryptography,
  trust-center/independently-audited)
- official blog posts: https://nym.com/blog/lewes-protocol-post-quantum (2026-04-20),
  https://nym.com/blog/mixnet-tuning (2026-07-14), https://nym.com/blog/nyms-zero-knowledge-network
  (2024-10-31)

## namada

- repository: https://github.com/namada-net/namada (https://github.com/anoma/namada redirects
  there), commit `024ff13d43983575f40d87257e6a3922fe9a094f` (2026-09-10)
- docs and specs: https://github.com/anoma/namada-docs, commit
  `ede866609a96776800ef574c8a6ae5cb0a9c1816` (2025-12-02); rendered at https://docs.namada.net and
  https://specs.namada.net
- genesis: https://github.com/namada-net/namada-mainnet-genesis (readme, `MAINNET-GUIDE.md`,
  `genesis/parameters.toml`, `genesis/tokens.toml`)
- masp crates: https://github.com/anoma/masp (readme)
- files cited: `README.md`, `Cargo.toml`, `audits/report-anoma-inference.pdf`,
  `crates/{core,tx,token,shielded_token,ibc,node,sdk,apps,ethereum_bridge}/**` as named in the pages,
  `wasm/Cargo.toml`, `packages/docs/pages/**` and `packages/specs/pages/**` in namada-docs
- website: https://namada.net (home, mainnet-launch, security, trusted-setup, roadmap)
- official blog posts under https://namada.net/blog/: namada-mainnet-is-live (2024-12-03),
  namada-mainnet-launch-is-complete (2025-06-20), completion-of-the-namada-trusted-setup,
  announcing-the-namada-trusted-setup-ceremony, understanding-the-masp-and-cc-circuits,
  shielding-the-multichain-how-namadas-data-protection-works,
  shielded-swaps-on-osmosis-an-upcoming-leap-in-privacy-via-namada, namada-roadmap

## tooling

- https://github.com/alinaschanz/repository-history-tools, run with default options against each
  clone on 2026-09-15; the numbers in provenance.md come from its `summary.json` files
