# Common Weights catalog

The reference catalog of [Common Weights](https://commonweights.org): open models, each revision
described by a signed manifest that `cw` pulls peer to peer and checks byte for byte.

The catalog holds no weights. It holds what is needed to find and verify them: the files of every
revision with their SHA-256 and Merkle roots, the torrent built from them, signed claims about who
checked them, and signed refs such as `main`.

## Use it

Clone it and point `cw` at the folder:

```
git clone https://github.com/commonweights/catalog
cw catalog add ./catalog
cw pull Qwen/Qwen3-8B
```

`git pull` brings new entries. The folder is also a static site: served as it is, `cw catalog add
<its URL>` works the same way.

## Layout

| Path | What |
|---|---|
| `catalog.json` | Format version and the denylists this catalog applies |
| `models/Org/Name.json` | One model: its revisions, their roots and attestations, and where its refs live |
| `manifests/Org/Name/<revision>.json` | The manifest of one revision: every file, its size, SHA-256 and Merkle root |
| `manifests/Org/Name/<revision>.torrent` | The BitTorrent v2 torrent of that revision, derived from the manifest |
| `refs/Org/Name.json` | The signed log of the model's refs (`main` and others) |
| `index.json`, `index.html` | Generated from the above, for browsing and for clients that read one file |
| `lists/` | The lists of hub repositories the indexer was run on |

Everything except `lists/` is written by `cw` and is deterministic: indexing the same revision again,
or merging another node's export, gives the same bytes.

## How entries get here

`cw index` streams every file of a Hugging Face repository at a fixed commit, hashes it on the way
and keeps nothing but the manifest. Large files are checked against the SHA-256 the hub records.
Each manifest is then signed with an `index` attestation, and each model's `main` is moved to the
indexed commit:

| Issuer | Key |
|---|---|
| `commonweights` | `ed25519:1944f265c1654ff91549e71bc014686e5f283b9eab48b863a56a7969a7c52112` |

An `index` attestation claims only that these are the bytes the hub served for that commit on that
date. It says nothing about the model itself; the license of each model is its own.
