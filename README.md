# kmapper-iscc-scan

Workspace-based ISCC content inventory and similarity clustering tool.

## How it works

This tool is built on the [ISCC (International Standard Content Code)](https://iscc.codes), an ISO standard (ISO 24138) for content-derived, decentralized media identifiers.

**Scanning** walks a directory recursively and generates an ISCC for each supported file. The ISCC encodes information about the file's content — not just its name or hash — so two files with different names but identical content will produce the same code. Each result is written as a sidecar `.iscc.json` file next to (or mirroring) the original file in the workspace.

**Compiling** aggregates all sidecar files from all scans into a single CSV inventory. It uses the Content Unit embedded in each ISCC to cluster files by similarity via Hamming distance. The result lets you identify:

- **Exact duplicates** — same content, regardless of filename
- **Near-duplicates and similar content** — e.g. a `.pptx` presentation and its `.pdf` handout will appear in the same cluster

## Installation

### 1. Check your Python version

```bash
python3 --version
```

You need **Python 3.12 or higher**. If the command is not found or the version is too old, see below.

<details>
<summary>Installing Python</summary>

**macOS / Linux:** Download from [python.org](https://www.python.org/downloads/) or use your system package manager.

**Windows:** The easiest option is [uv](https://docs.astral.sh/uv/), a fast Python manager written in Rust:

```powershell
# Install uv
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"

# Install Python 3.12
uv python install 3.12
```

</details>

### 2. Install the package

```bash
pip install kmapper-iscc-scan
```

## Usage

```bash
# Scan a directory into a workspace
kmapper-iscc-scan scan /path/to/content /path/to/workspace

# Optionally scan with a custom named batch
kmapper-iscc-scan scan /path/to/content /path/to/workspace --batch myBatch

# Compile an inventory CSV with similarity clustering
kmapper-iscc-scan compile /path/to/workspace # This uses a default Hamming distance of 10 (e.g. approx. 84.38%)

# Optionally compile with your own threshold for the Hamming distance
kmapper-iscc-scan compile /path/to/workspace --threshold 15

# Optionally compile with your own similarity threshold given in percent
kmapper-iscc-scan compile /path/to/workspace --similarity 90

# Show workspace status
kmapper-iscc-scan status /path/to/workspace
```

## Workspace structure

```
workspace/
  scan_log.json
  sidecars/
    my-batch/
      subdir/
        file.pdf.iscc.json
  iscc_inventory.csv
```

## License

MIT
