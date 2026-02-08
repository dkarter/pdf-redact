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
- Verification step to ensure terms are not present in extracted text

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

## CLI options

- `--term <value>`: term to redact (repeatable)
- `--regex-term <pattern>`: regex pattern to redact (repeatable)
- `--terms-file <path>`: rules file with one term per line
- `--source <path>`: PDF file path or directory path (required)
- `--output-path <dir>`: output directory for redacted PDFs (required)
- `-r, --recursive`: recursively scan source directory for PDFs
- `--case-insensitive`: case-insensitive term matching
- `--dry-run`: scan only, do not write files

At least one of `--term`, `--regex-term`, or `--terms-file` is required.

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
- A verification pass checks that terms are absent from extracted text.

No automated redaction tool can guarantee semantic perfection for every PDF structure, so spot-check output visually and with text extraction when handling sensitive records.
