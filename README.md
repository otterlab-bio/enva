# enva

**A rattler-first environment manager for reproducible bioinformatics workflows.**

enva creates and maintains native rattler environments while discovering and interoperating with existing `conda`, `mamba`, and `micromamba` environments when explicitly requested.

## Proof

`enva list` resolves every accessible environment to a concrete prefix and attributes it to an owner:

```text
$ enva list
Name                           | Owner      | Prefixes
-------------------------------------------------------------------------------
bamdriver-gate-a-local-verify  | rattler    | <rattler-root>/envs/bamdriver-gate-a-local-verify
pairbam-gate-b-local-verify    | rattler    | <rattler-root>/envs/pairbam-gate-b-local-verify
quarto-env                     | rattler    | <rattler-root>/envs/quarto-env
```

The `Owner` column is the whole point of the compatibility model below: an environment enva created
reports as `rattler`, while one discovered from conda, mamba, or micromamba keeps reporting as
external until it is adopted.

## Why enva

- Native create, solve, install, run, validate, and remove operations through rattler.
- Explicit adoption of external environments instead of implicit mutation.
- Deterministic handling of duplicate names through `--prefix`.
- JSON output, dry-run validation, detailed listings, cache cleanup, and shell integration.
- Built-in Otter environments: `otter-core`, `otter-snakemake`, and `otter-extra`.

## Install

Download the release binary for your platform, or build from source:

```bash
git clone https://github.com/otterlab-bio/enva.git
cd enva
cargo build --release
./target/release/enva --help
```

## First use

Create and inspect the standard runtime environments:

```bash
enva create --all
enva list --detailed
enva validate --all
enva run otter-core -- fastqc --version
```

Create a custom environment:

```bash
enva create \
  --yaml ./src/configs/otter-core.yaml \
  --name otter-core
```

Add packages using separate MatchSpec arguments:

```bash
enva install --name otter-core fastqc seqkit
enva install --name otter-core 'numpy>=1.24,<2'
```

## Shell integration

```bash
eval "$(enva shell hook bash)"
enva activate otter-core
enva deactivate
```

Equivalent one-shot activation is available through `eval "$(enva activate otter-core)"`. Fish and PowerShell hooks are also supported.

## Compatibility model

| Operation | Native rattler | Explicit compatibility path |
| --- | --- | --- |
| Create and cache cleanup | Yes | Delegated when selected |
| YAML validation | Native solve | Basic delegated validation |
| Install/remove | Rattler-owned prefixes | Adopted/external prefixes only |
| Discover/list/run | Native registry | conda/mamba/micromamba discovery |
| Adopt external prefix | Yes | Unsupported |

Important rules:

- `ENVA_BACKEND=cli` is an expert compatibility mode; the default remains rattler-first.
- `micromamba` is never downloaded by enva and must already be installed or configured through `ENVA_MICROMAMBA_PATH`.
- Multiple accessible environments with the same name fail closed until `--prefix` selects one.
- External environments must be adopted explicitly before rattler mutation or removal.
- `pip:` subsections in environment YAML are intentionally rejected by the rattler backend.

## Development

```bash
cargo fmt --all -- --check
cargo test
cargo build --release
```

The full end-to-end workflow also exercises standard environment creation, mixed-channel package installation, adopted micromamba prefixes, replacement under an active `CONDA_PREFIX`, and removal safeguards.

## License

MIT
