# kmapper-iscc-scan

Workspace-based ISCC content inventory and similarity clustering tool.

## Installation

```bash
pip install kmapper-iscc-scan
```

## Usage

```bash
# Scan a directory into a workspace
kmapper-iscc-scan scan /path/to/content /path/to/workspace

# Optionally scan with a named batch
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
