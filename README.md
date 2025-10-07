# v86-fs

A composite GitHub Action that creates a v86 filesystem from a source directory.

## Overview

This action clones the [copy/v86](https://github.com/copy/v86) repository and runs the filesystem creation scripts to produce a tarball containing the v86 filesystem artifacts (`fs.json` and `base/` directory).

## Usage

```yaml
- name: Create v86 filesystem
  uses: arcanis/v86-fs@main
  with:
    source: /path/to/source/directory
    output: /path/to/output/directory
```

## Inputs

### `source` (required)

The root directory of the virtual filesystem that will be converted to v86 format.

### `output` (required)

The path where the v86 folder will be stored. The action will create:
- `fs.json` - The filesystem metadata
- `base/` - Directory containing SHA256-named file chunks
- `v86-fs.tar.gz` - Tarball containing both `fs.json` and `base/`

## Example

```yaml
name: Build v86 Filesystem
on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
      
      - name: Prepare source filesystem
        run: |
          mkdir -p /tmp/my-fs
          echo "Hello from v86!" > /tmp/my-fs/hello.txt
      
      - name: Create v86 filesystem
        uses: arcanis/v86-fs@main
        with:
          source: /tmp/my-fs
          output: ./output
      
      - name: Upload artifacts
        uses: actions/upload-artifact@v3
        with:
          name: v86-filesystem
          path: output/v86-fs.tar.gz
```

## How It Works

The action performs the following steps:

1. Clones the [copy/v86](https://github.com/copy/v86) repository to get the filesystem creation tools
2. Runs `fs2json.py` to create the filesystem metadata JSON from the source directory
3. Runs `copy-to-sha256.py` to copy files with SHA256-based naming to the `base/` directory
4. Creates a tarball containing both the JSON metadata and the base directory
5. Cleans up temporary files

## References

- [v86 Filesystem Documentation](https://github.com/copy/v86/blob/master/docs/filesystem.md)
- [copy/v86 Repository](https://github.com/copy/v86)