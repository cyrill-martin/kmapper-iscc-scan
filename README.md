# kmapper-iscc-scan

Workspace-based ISCC content inventory and similarity clustering CLI.

Developed by [kmapper GmbH](https://kmapper.ch), not related to the k-means mapper algorithm from topological data analysis.

## How it works

This CLI is built on the [ISCC (International Standard Content Code)](https://iscc.codes), an ISO standard (ISO 24138) for content-derived, decentralized media identifiers.

**Scanning** walks a directory recursively and generates an ISCC for each supported file. The ISCC encodes information about the file's content (not just its name or hash) so two files with different names but identical content will produce the same code. Each result is written as a sidecar `.iscc.json` file next to (or mirroring) the original file in the workspace.

**Compiling** aggregates all sidecar files from all scans into a single CSV inventory. It uses the Content Unit embedded in each ISCC to cluster files by similarity via Hamming distance. The result lets you identify:

- **Exact duplicates** — same content, regardless of filename
- **Near-duplicates and similar content** — e.g. a `.pptx` presentation and its `.pdf` handout will appear in the same cluster

## Installation

### 1. Check your Python version

```bash
python --version
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

### Scanning

Scan a directory recursively (i.e. including all subdirectories) to generate an ISCC for each relevant file and store the metadata files in your given workspace directory:

```bash
# Scan a directory into a workspace
kmapper-iscc-scan scan /path/to/content /path/to/workspace
```

The above command will create a default batch name for your scanned directory. You can optionally determine your own batch name with:

```bash
kmapper-iscc-scan scan /path/to/content /path/to/workspace --batch my-batch
```

### Compiling

Compile all the metadata files from all your different scans into one inventory CSV file, including clustering of identical or similar files:

```bash
# Compile an inventory CSV with similarity clustering
kmapper-iscc-scan compile /path/to/workspace
```

The above command will use a default Hamming distance of 10 (i.e. approx. 84.38% similarity). You can optionally set your own Hamming distance or indicate a similarity in percent:

```bash
# Optionally compile with your own Hamming distance
kmapper-iscc-scan compile /path/to/workspace --hamming 15

# Optionally compile with your own similarity in percent
kmapper-iscc-scan compile /path/to/workspace --similarity 90
```

#### The inventory CSV

The CSV contains one row per file. The three grouping columns follow a hierarchy — each level is a subset of the one below:

- **`instance_group`** — files that are byte-for-byte identical (exact copies, regardless of filename or location). Files in the same instance group are always also in the same data group.
- **`data_group`** — files with the same data structure (e.g. the same PDF re-saved with slightly different metadata, causing the raw bytes to differ). Files in the same data group are always also in the same content cluster.
- **`content_cluster`** — files with similar content regardless of format or encoding (e.g. a `.pptx` presentation and its `.pdf` handout). This is the broadest grouping.

The following example shows a selection of columns to illustrate the grouping. The actual CSV also includes batch, paths, filesize, ISCC code, and the individual ISCC units.

| filename                   | mediatype | instance_group | data_group | content_cluster | hamming_threshold | similarity_threshold |
| -------------------------- | --------- | -------------- | ---------- | --------------- | ----------------- | -------------------- |
| demo.pptx                  | pptx      | 1              | 1          | 1               | 10                | 84.38                |
| demo_final.pptx            | pptx      | 1              | 1          | 1               | 10                | 84.38                |
| document_v1.docx           | docx      |                |            | 1               | 10                | 84.38                |
| version_basel_on-site.docx | docx      |                |            | 1               | 10                | 84.38                |
| final_print-version.pdf    | pdf       |                |            | 1               | 10                | 84.38                |
| screenshot.png             | png       | 2              | 2          | 2               | 10                | 84.38                |
| screenshot_jira_v2.png     | png       | 2              | 2          | 2               | 10                | 84.38                |

`demo.pptx` and `demo_final.pptx` are byte-for-byte identical despite their different names. Together with the `.docx` and `.pdf` files they form a content cluster of similar documents. The two screenshots are exact copies in a separate cluster.

### Checking the Scans

Check which directories have already been scanned:

```bash
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
