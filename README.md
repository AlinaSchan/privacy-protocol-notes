# privacy protocol notes

notes on five systems that hide something a public blockchain would show: darkfi, penumbra, anoma,
nym and namada. one page per protocol, then comparisons of what an outside observer can see, how the
privacy models differ, what each system assumes about identity, what leaks at the network layer, and
what an outsider can verify.

the question behind the repository: what can an outside observer actually see?

this is not a ranking. the five solve different problems. three are chains with shielded state
(darkfi, penumbra, namada), one is an architecture for intent-based applications built on a resource
model (anoma), and one is a mixnet with a chain attached for payments and the node registry (nym).
where a comparison does not hold, the page says so instead of forcing a row.

not compared: throughput, token, market capitalization, funding, team size.

## pages

- [protocols/darkfi.md](protocols/darkfi.md), [penumbra.md](protocols/penumbra.md),
  [anoma.md](protocols/anoma.md), [nym.md](protocols/nym.md), [namada.md](protocols/namada.md):
  one page each, same headings, so the comparisons can point back to them
- [comparisons/observable-state.md](comparisons/observable-state.md): public state, private state,
  what can be derived
- [comparisons/privacy-models.md](comparisons/privacy-models.md): the primitives, the trusted setup
  question, what each model is designed to hide and from whom
- [comparisons/identity.md](comparisons/identity.md): accounts, keys, viewing keys, validator and
  node identity
- [comparisons/network-metadata.md](comparisons/network-metadata.md): what leaks before anything
  reaches a chain
- [comparisons/provenance.md](comparisons/provenance.md): what an outsider can check, and how each
  project documents itself
- [sources.md](sources.md): the repositories, commit hashes, documentation and papers used

## method

primary sources only: each project's repository, its documentation and specification (mostly kept
in the same repository), its papers, and its website for what it says about itself. i cloned the
repositories on 2026-09-15; the commit hashes are in sources.md, and every protocol page repeats the
hash it was written against. every row in a comparison names the file it comes from. where two
official sources disagree, both are quoted. where i could not confirm something, the entry says
"not clear from the public material".

the darkfi page condenses a longer set of notes,
[darkfi-protocol-notes](https://github.com/alinaschanz/darkfi-protocol-notes). the repository
statistics come from
[repository-history-tools](https://github.com/alinaschanz/repository-history-tools).

## disclaimer

independent notes. i am not affiliated with any of these projects and i speak for none of them.
interest is not endorsement, and none of this is advice about any token.
