# go-yaml

Fork of `yaml/go-yaml` v3 on branch `lenient-aliases`. Adds `UnmarshalLenient` and `UnmarshalStubAliases` for parsing and round-tripping YAML files that reference anchors defined in other files. Module: `github.com/modopayments/go-yaml/v3`.

## Upstream

- `origin` → `yaml/go-yaml` (upstream)
- `modopayments` → `modopayments/go-yaml` (our fork)
- `fork` → `W-Floyd/go-yaml` (personal fork, ignore)

## Build & test

```sh
go test ./...
```

## Key files

- `decode.go` — `UnmarshalLenient` (lenient anchor resolution), `UnmarshalStubAliases` (alias stubbing), `lenientAliases`/`stubAliases` parser flags
- `encode.go` — `AliasNode` re-encoding (emits `*anchorName` from `node.Value` without requiring `node.Alias`)
- `scannerc.go` — `preserve_plain_multiline` flag (preserves internal newlines in plain scalars)

## Modo additions (5 commits on top of upstream v3)

1. `UnmarshalLenient` — cross-file anchor tolerance (missing anchors → null)
2. Module rename to `github.com/modopayments/go-yaml/v3`
3. `UnmarshalStubAliases` — alias stubs for lossless round-trip (+ nil-guard in `d.alias()`)
4. `preserve_plain_multiline` — plain multi-line scalars keep their newlines
5. MODO.md

## Important behaviors

- `UnmarshalStubAliases` is only correct with `*yaml.Node` output. With struct/map output, stubbed alias nodes silently decode as zero values.
- `UnmarshalStubAliases` stubs in-file anchors too (intentional — required for consistent round-trip).
- Multi-line plain scalars re-encode as `|` block scalars after a stub round-trip (content unchanged, style changes).

## Staying in sync with upstream

```sh
git fetch origin
git rebase origin/v3
```

Conflicts are typically in `decode.go` and `scannerc.go`.
