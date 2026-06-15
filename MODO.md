# go-yaml — Modo fork

Modo's fork of [go-yaml/go-yaml](https://github.com/go-yaml/go-yaml) (v3), extended with two features required for lossless round-tripping of composition files that use YAML anchors defined across multiple files.

---

## Changes from upstream

### `UnmarshalLenient` — cross-file anchor tolerance
```go
func UnmarshalLenient(in []byte, out interface{}) error
```
Like `Unmarshal` but silently replaces any unresolved anchor reference with `null` instead of returning an error. Useful when parsing a single composition file that references anchors defined in a sibling file (e.g. a shared `_anchors.yaml`).

### `UnmarshalStubAliases` — lossless round-trip with cross-file anchors
```go
func UnmarshalStubAliases(in []byte, out interface{}) error
```
Parses the document into a `*yaml.Node` tree but leaves every alias reference as an unresolved `AliasNode` (with `Alias == nil` and `Value` holding the anchor name). Neither in-file nor cross-file anchors are followed.

The resulting node tree can be re-encoded without modification: the encoder emits `*anchorName` using only `node.Value`, so a `nil Alias` is safe. This enables `composition-lsp` and `composition-fmt` to read, modify jq block scalars, and write back composition files that contain cross-file anchor references without corrupting them.

Multi-line plain scalars preserve their internal newlines when re-encoded (upstream collapses them to spaces).

---

## Why a fork

Upstream `go-yaml` v3 returns an error for unresolved anchor references, making it impossible to parse or round-trip composition files that split anchors across files. The two new functions are narrow additions that do not affect existing behaviour.

---

## Module path

```
github.com/modopayments/go-yaml/v3
```

Used by `composition-lsp` for YAML parsing and `composition-fmt` for lossless reformatting.
