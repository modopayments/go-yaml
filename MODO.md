# go-yaml — Modo fork

Modo's fork of [go-yaml/go-yaml](https://github.com/go-yaml/go-yaml) (v3), extended with two functions required for lossless round-tripping of composition files that use YAML anchors defined across multiple files.

Module path: `github.com/modopayments/go-yaml/v3`
Branch: `lenient-aliases` (5 commits ahead of upstream v3)

---

## Changes from upstream

### `UnmarshalLenient` — cross-file anchor tolerance

```go
func UnmarshalLenient(in []byte, out interface{}) error
```

Like `Unmarshal` but silently replaces any unresolved anchor reference (`*anchorName` with no matching `&anchorName` in the same byte slice) with `null` instead of returning an error. Useful when parsing a single composition file that references anchors defined in a sibling file (e.g. a shared `_anchors.yaml`).

In-file anchors (defined in the same byte slice) still resolve normally.

**Limitations:**
- The original anchor name is not preserved in the substituted null node. Callers cannot enumerate which anchors were missing post-hoc.
- Not available on `*Decoder` — there is no `(*Decoder).DecodeLenient()`. Lenient mode requires `UnmarshalLenient`.

### `UnmarshalStubAliases` — lossless round-trip with cross-file anchors

```go
func UnmarshalStubAliases(in []byte, out interface{}) error
```

Intended for `*yaml.Node` output only. Instead of resolving aliases, keeps every alias (both in-file and cross-file) as an unresolved `AliasNode` stub with `Alias == nil` and `Value` holding the anchor name. Neither in-file nor cross-file anchors are followed.

The resulting node tree can be re-encoded without modification: the encoder emits `*anchorName` using only `node.Value`, so a nil `Alias` is safe. This enables `composition-lsp` and `composition-fmt` to read, modify jq block scalars, and write back composition files that contain cross-file anchor references without corrupting them.

**What is preserved:** all scalar values, mapping keys, sequence elements, anchor definitions (`&name`), alias syntax (`*name`), comments, and document structure.

**What changes on re-encode:** Multi-line plain scalars are re-emitted as literal block scalars (`|`). The `preserve_plain_multiline` flag (added by this fork) converts internal fold-spaces to real newlines; the encoder then promotes these scalars to `LiteralStyle`. Content is byte-identical; the YAML style changes from plain to block.

**Important limitation:** When `out` is a struct or map (not `*yaml.Node`), alias-stubbed nodes decode as zero values for their keys with no error or warning. `UnmarshalStubAliases` is only useful for `*yaml.Node` round-trips — do not use it if you need to read values from aliased nodes.

**Note on in-file anchors:** `UnmarshalStubAliases` also stubs in-file anchors (intentional, for round-trip consistency). If an alias references an anchor defined earlier in the same file and you need to read that value, use `Unmarshal` instead.

---

## Why a fork

Upstream `go-yaml` v3 returns an error for unresolved anchor references, making it impossible to parse or round-trip composition files that split anchors across files. The two new functions are narrow additions that do not affect existing behaviour.

---

## Module path

```
github.com/modopayments/go-yaml/v3
```

Used by `composition-lsp` for YAML parsing and `composition-fmt` for lossless reformatting.
