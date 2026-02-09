# pdf-redact

Safe PDF text redaction CLI using PyMuPDF. It removes matched text with native PDF redaction (not a visual overlay), then rewrites and verifies output.

## Features

- Redact one or more terms with repeated `--term` flags
- Redact regex patterns with repeated `--regex-term` flags
- Load mixed exact and regex rules from a terms file
- Process a single PDF or a directory of PDFs
- Optional recursive directory scan
- Optional case-insensitive matching
- Dry-run mode to preview match counts
- Verification step with configurable `warn`/`fail` behavior when matches remain

## Prerequisites

- Python 3.12+ (this repo uses mise)

```bash
mise use python@3.12
```

## Install dependencies

Using pip:

```bash
python -m pip install -r requirements.txt
```

Using uv:

```bash
uv pip install -r requirements.txt
```

## Install as a global CLI

Using uv directly from a git repo:

```bash
uv tool install "git+https://github.com/dkarter/pdf-redact.git"
```

Using mise from GitHub releases (published binaries):

```bash
mise use -g "github:dkarter/pdf-redact"
```

Using uv from a local checkout:

```bash
uv tool install .
```

After install, verify:

```bash
pdf-redact --help
```

## Usage

Single file:

```bash
./pdf-redact \
  --term "Jane Smith" \
  --regex-term "\\b45D01-[0-9A-Z-]+\\b" \
  --term "123 Main St" \
  --source "./sources/input.pdf" \
  --output-path "./out"
```

Directory of PDFs:

```bash
./pdf-redact \
  --term "Jane Smith" \
  --term "123 Main St" \
  --source "./sources" \
  --output-path "./out" \
  -r
```

Dry run (no output files written):

```bash
./pdf-redact \
  --term "Jane Smith" \
  --source "./sources" \
  --output-path "./out" \
  --dry-run
```

Verification warning mode (default: keep output file, print warning):

```bash
./pdf-redact \
  --term "Jane Smith" \
  --source "./sources/input.pdf" \
  --output-path "./out" \
  --verify-on-match warn
```

Verification strict mode (fail file when verification still matches):

```bash
./pdf-redact \
  --term "Jane Smith" \
  --source "./sources/input.pdf" \
  --output-path "./out" \
  --verify-on-match fail
```

## CLI options

- `--term <value>`: term to redact (repeatable)
- `--regex-term <pattern>`: regex pattern to redact (repeatable)
- `--terms-file <path>`: rules file with one term per line
- `--source <path>`: PDF file path or directory path (required)
- `--output-path <dir>`: output directory for redacted PDFs (required)
- `-r, --recursive`: recursively scan source directory for PDFs
- `--case-insensitive`: case-insensitive term matching
- `--dry-run`: scan only, do not write files
- `--verify-on-match <warn|fail>`: verification behavior when terms still appear (`warn` prints warning and continues, `fail` marks file as failed)

At least one of `--term`, `--regex-term`, or `--terms-file` is required.

## Shell completions (usage)

This repo includes a usage spec at `pdf-redact.usage.kdl`.

The CLI can print embedded completion scripts directly:

```bash
pdf-redact completion zsh
pdf-redact completion bash
pdf-redact completion fish
```

Install `usage`:

```bash
mise use -g usage
```

Generate completions:

```bash
usage g completion bash pdf-redact -f ./pdf-redact.usage.kdl > ~/.bash_completions/pdf-redact.bash
usage g completion zsh pdf-redact -f ./pdf-redact.usage.kdl > ~/.zsh_completions/pdf-redact.zsh
usage g completion fish pdf-redact -f ./pdf-redact.usage.kdl > ~/.config/fish/completions/pdf-redact.fish
```

Note: generated completion scripts call `usage` at runtime, so `usage` must be installed on machines using these completions.

## Terms file format

Each non-empty line is one rule. Lines starting with `#` are comments.

- Exact match term: `Jane Smith`
- Regex term with prefix: `re:\b45D01-[0-9A-Z-]+\b` or `regex:\d{3}-\d{2}-\d{4}`
- Regex term with slashes: `/\b[A-Z]{2}\d{6}\b/i` (supports `i`, `m`, `s` flags)

Example `terms.txt`:

```text
# exact
Jane Smith

# regex
re:\b\d{3}-\d{2}-\d{4}\b
/\b45D01-[0-9A-Z-]+\b/i
```

## Safety notes

- Input files are never modified in place.
- Output is written to a temp file and atomically moved into place.
- For matched files, save uses full rewrite settings (`incremental=False`, garbage collection, stream cleanup).
- A verification pass checks extracted text and raw bytes in output PDFs.
- `--verify-on-match warn` (default) prints warnings and still writes output.
- `--verify-on-match fail` treats verification hits as file failures.

No automated redaction tool can guarantee semantic perfection for every PDF structure, so spot-check output visually and with text extraction when handling sensitive records.

## Release automation

- `release-please` runs on `main` and opens/updates release PRs from conventional commits.
- When a release is published, GitHub Actions builds PyInstaller binaries for Linux, macOS (x64 + arm64), and Windows.
- Assets are uploaded with names like `pdf-redact-v1.2.3-darwin-arm64.tar.gz`, which the mise GitHub backend can detect.
- `release-please` should run with a PAT stored as `RELEASE_PLEASE_TOKEN` (repo secret) so release events can trigger downstream workflows.

## License

This project is licensed under the GNU Affero General Public License v3.0 or later (`AGPL-3.0-or-later`).
See `LICENSE` for the full text.
