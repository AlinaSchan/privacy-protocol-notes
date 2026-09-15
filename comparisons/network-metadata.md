# network metadata

what leaks before a transaction ever reaches the chain, and what the network learns while a wallet
keeps up with it. citation keys as in [observable-state.md](observable-state.md); full list in
[sources.md](../sources.md).

nym's whitepaper states the problem for all five: peer-to-peer broadcast "expose[s] ip addresses,
timing, and other network metadata" no matter what proofs the chain carries (whitepaper p.1). the
other four answer it in different places, or not at all.

## the transport between a user and the network

| protocol | how a transaction travels | built-in protection | what the docs recommend | source |
|---|---|---|---|---|
| darkfi | the wallet talks to a `darkfid` over local rpc; the node relays through the project's own p2p library | transports tcp, tls, tor, i2p, nym, socks5, quic, unix; per-transport profiles so a tor node does not dial clearnet seeds; no mixing or cover traffic in the p2p layer | run over tor, i2p or nym; "subject to that transport's threat model" | `darkfi:src/net/transport/`, `doc/src/dep/0007.md`, `doc/src/start-here.md` |
| penumbra | the client plans and proves locally, then broadcasts through a node's grpc | none; cometbft gossip on 26656/tcp | the fuzzy message detection threat model assumes tor and https between user and detector | guide join-network page, `penumbra:SPEC/crypto/fmd/threat_model.md` |
| anoma (anomapay) | a signed intent goes to a transaction service, then a proving service, then the service submits to the adapter | none; the services see the witness | nothing found | docs.anoma.net anoma-pay-architecture, anoma-pay-trust-model |
| nym | the packet is the product: sphinx packets over five hops, or two wireguard hops in dvpn mode | per-hop mixing with a 15 ms mean delay, fixed 2,413-byte packets, loop cover traffic, single-use reply blocks | choose the mode for the threat; a fixed exit "behaves like a single dvpn exit" against the destination | `nym:D/network/mixnet-mode.mdx`, `D/network/deep-dives/packet-anatomy.mdx`, `documentation/docs/lib/privacy-model/examples/generic.ts` |
| namada | the client submits to a cometbft rpc | none | "tor/vpn" on the privacy page; "verify that the operator can be trusted" for public rpcs | `ndocs:docs/introduction/privacy.mdx`, `docs/introduction/public-rpc.mdx` |

## what a peer or a server learns

| protocol | learns | source |
|---|---|---|
| darkfi | a peer: transport address, app name, protocol version, clock, advertised inbound addresses, features, an optional node id "only used for debugging. compromises privacy when set"; whitelists with `last_seen` stamps travel between peers | `darkfi:src/net/message.rs`, `doc/src/arch/net/p2p-network.md` |
| penumbra | the view server holds the full viewing key and "does not support any kind of transport security"; a node sees which heights a client fetches full transactions for, since the client fetches only blocks with detected activity ("chaff downloads" are a todo) | `penumbra:crates/bin/pclientd/src/lib.rs`, `crates/view/src/worker.rs` |
| anoma | the transaction service and the proving service see sender, receiver, asset and amount; the proving operator sees the unencrypted witness "temporarily"; the indexer knows a discovery key per user | docs.anoma.net anoma-pay-trust-model, key-management |
| nym | the entry gateway: the client's ip, identity key, ticket serials, bandwidth; the exit: destination ip and port and unencrypted payload; the network requester: the client's nym address on every connect unless anonymous replies are on; the mix nodes: one hop each | `nym:D/network/infrastructure/exit-services.mdx`, `D/network/infrastructure/nym-nodes.mdx`, `common/gateway-requests/src/registration/handshake/messages.rs` |
| namada | the rpc or masp indexer: the height ranges a wallet asks for, bounded by a key's birthday; the web wallet depends on an rpc, an indexer and a masp indexer "operated by the same provider" | `namada:crates/sdk/src/masp/utilities.rs`, `ndocs:docs/users/shielded-accounts/masp-keys.mdx`, `docs/users/namadillo/overview.mdx` |

## timing and size

- darkfi: sizes and timing are whatever the transport gives; the p2p page lists rate limits and
  scoring as design notes, not as privacy measures. (`darkfi:doc/src/arch/net/p2p-network.md`)
- penumbra: transactions carry an expiry height and a fixed number of detection clues equal to the
  number of outputs, with dummies, as a consensus rule; ibc timeouts are quantized to the minute.
  (`penumbra:SPEC/crypto/fmd/system_mapping.md`, `PROTO/core/component/ibc/v1/ibc.proto`)
- anoma: the host chain's block times; nothing on the adapter path.
- nym: packets are fixed size in mixnet mode; dvpn mode passes inter-packet timing unchanged and
  does not pad; the two modes are "distinguishable at the protocol level"; a global observer's
  bulk-flow correlation is "an open question". (`nym:D/network/dvpn-mode/protocol.mdx`,
  `documentation/docs/lib/privacy-model/examples/generic.ts`)
- namada: transactions carry a timestamp and an expiration; the default client expiration is one
  hour. (`namada:crates/tx/src/section.rs`, `ndocs:docs/users/transparent-accounts/multisignature.mdx`)

## what the mixnet hides that a shielded ledger does not

a shielded ledger, once the transaction is in a block, hides values and parties from everyone who
reads the chain. it does not hide who handed the transaction to the first node, from which address,
at what time, or which blocks a wallet later asks about. nym's design is aimed at exactly that gap
and at nothing else: its own docs say "no amount of mixing protects you against the server you are
talking to", and that against the destination the mixnet "is equivalent to dvpn"
(`nym:D/network/threat-model/two-layer-model.mdx`, `D/network/mixnet-mode.mdx`).

the reverse also holds. a mixnet carries bytes; if the bytes are a transparent transaction, the
chain still publishes the amount and the parties. darkfi is the one of the four ledgers that ships
a nym transport in its own node (`darkfi:src/net/transport/nym.rs`); whether the testnet is
reachable through it today is not clear from the public material.

## open questions across the five

- how darkfi nodes relay transactions to each other (the p2p page has suggestions, not a protocol
  description). (`darkfi:doc/src/arch/net/p2p-network.md`)
- whether penumbra's view protocol will get the "chaff downloads" its worker marks as a todo.
  (`penumbra:crates/view/src/worker.rs`)
- whether anoma intents are gossiped between nodes anywhere in production, or only sent to a
  service. (docs.anoma.net anoma-pay-architecture)
- whether nym mix nodes generate their own cover traffic in the current node code; the docs say
  yes, i found no generator in `nym-node`. (`nym:D/network/mixnet-mode/cover-traffic.md`)
- what a namada masp indexer can infer from a wallet's note-range requests; no threat-model page
  exists in the docs tree. (`ndocs:docs/users/shielded-accounts/shielded-sync.mdx`)
