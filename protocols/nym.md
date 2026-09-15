# nym

a mixnet for network-level privacy, with a cosmos sdk chain (nyx) that holds the node registry,
stake, rewards and credential deposits. written against the nym monorepo at commit `60f499eb`
(2026-09-15). citation key: `D` is `documentation/docs/pages/` in the monorepo, the source of
nym.com/docs; `WP` is the 2021 whitepaper.

## in its own words

"decentralised privacy infrastructure that protects against network-level surveillance"; it
protects "who talks to whom, when, how often, and how much" (`D/network/index.md`).

## status

the tag `nym-binaries-1.0.0` and the commit "set mainnet as the default network build target" are
both dated 2022-05-03; the latest release in the changelog is `2026.17-djibouti`, 2026-09-01. two
traffic modes: dvpn (two wireguard hops, no mixing) and mixnet (five hops, sphinx packets, delays,
cover traffic), both in the nymvpn app; the sdks expose the mixnet (`D/network/index.md`). the
company is nym technologies sa, switzerland (nym.com/trust-center). libraries apache 2.0 or mit,
binaries gpl 3.0, docs cc by-nc-sa (`D/network/licensing.md`).

## public state

nym's chain is not where the privacy is; its state is mostly public by design:

- every bonded node: host, identity key, http port, bond, owner, operator cost, profit margin;
  delegations; the rewarded set chosen each hour (60 mix nodes, 80 entry and 100 exit gateways per
  the reward parameters json; another docs page gives 50/70/120) (`D/operators/nodes/nym-node/
  bonding.mdx`, `D/operators/tokenomics/mixnet-rewards.mdx`, `D/operators/tokenomics.mdx`)
- node performance annotations and gateway probe results, and each node's self-described data at
  its own http port (`D/operators/performance-and-testing.mdx`)
- credential deposits on the ecash contract: deposit id, amount, and a claimed identity key the
  contract does not verify; gateways' redemption proposals with ticket counts
  (`common/cosmwasm-smart-contracts/ecash-contract/src/msg.rs`)
- validators and voting power, nym balances and transfers, the dkg dealers and multisig
  membership (`D/apis/`)

## private state

what the mixnet hides: the client's ip from mix nodes, the exit and the destination; which packets
are cover; the full route; the sender of a reply sent through a single-use reply block; the link
from a deposit to a ticketbook to a ticket, and between tickets; the user's credential key
(`D/network/infrastructure/exit-services.mdx`, `D/network/mixnet-mode/cover-traffic.md`,
`D/network/cryptography/zk-nym/unlinkability.mdx`).

## identity assumptions

nodes hold an ed25519 identity and x25519 sphinx keys that rotate; they bond by signing a contract
message from a nyx wallet and keep a separate nyx account for config pushes (`nym-node/src/config/
persistence.rs`, `bonding.mdx`). clients hold an ed25519 identity and an x25519 key; a client
address embeds its entry gateway and is stable unless rotated (`D/network/reference/addressing.md`).
credentials start from a nyx address seen at deposit time, with the ed25519 key as a private
attribute; issuers are a quorum of nym-api signers drawn from validators, initial set 17, minimum
10 (`D/network/cryptography/zk-nym/zk-nym-overview.mdx`, `D/operators/nodes/validator-setup/
nym-api.mdx`). validator entry is whitelisted (`D/operators/faq/nyx-faq.mdx`).

## metadata exposure and network-level privacy

this is the protocol's whole subject, and its docs are specific about the limits:

- the whitepaper's starting point: peer-to-peer broadcast "expose[s] ip addresses, timing, and
  other network metadata" even when the chain has zero-knowledge proofs (`WP` p.1)
- the entry gateway sees the client ip; the exit sees destination ip and port and any unencrypted
  payload; the network requester receives the client's nym address on every connect unless
  anonymous replies are switched on (`exit-services.mdx`)
- the docs' threat model: mixing protects against network observers, and against the destination a
  fixed exit "behaves like a single dvpn exit"; protection against a global observer is graded
  partial, "resists per-packet correlation; long bulk flows weaken it"
  (`documentation/docs/lib/privacy-model/examples/generic.ts`, `D/network/threat-model/`)
- dvpn mode passes packet timing unchanged and does not pad packets; the two modes are
  "distinguishable at the protocol level" (`D/network/dvpn-mode/protocol.mdx`)
- sphinx packets are fixed at 2,413 bytes; per-hop delay defaults to a mean of 15 ms; a client
  sends about 50 packets per second, real or cover, plus loop cover (`D/network/deep-dives/
  packet-anatomy.mdx`, `common/client-core/config-types/src/lib.rs`)

## programmability

cosmwasm contracts on nyx (mixnet, vesting, ecash, dkg, multisig, directory, and others in
`contracts/`); rust and typescript sdks for applications that send through the mixnet; service
providers reachable only by nym address; an exit policy that is one deny list for all operators
(`D/developers/`, `exit-services.mdx`).

## provenance and what can be verified

open source; audits listed on the website: cure53 2024, oak security 2023, jp aumasson 2021, and
cryspen reviews "to be published" (nym.com/trust-center/independently-audited). anyone can run the
gateway probe and the node status api. a verifiable directory, where a client checks the node
directory against a state proof, landed in release 2026.17; the client core does not reference it
yet (`docs/directory/README.md`, `CHANGELOG.md`). no reproducible-build statement found.

## trust assumptions

the quorum of nym-api signers for credentials and for performance measurement; the whitelisted
validator set; honest entry gateways for ip and traffic accounting; the exit operator for content;
hardcoded roots in the client (directory checkpoint key, upgrade attester key, contract addresses)
(`common/network-defaults/src/mainnet.rs`).

## open questions

whether mix nodes emit their own cover traffic in the current node code (the docs say yes, i found
no generator); measured latency ("none have been measured", `D/developers/limitations.mdx`); the
number of validators and the current quorum; node and country counts, which three official pages
give differently; what the lewes protocol is, since the repository describes it as a gateway
registration protocol that reveals the client ip and the blog as post-quantum key exchange.
