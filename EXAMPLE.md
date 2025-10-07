# Usage Examples

## Basic Usage in GitHub Actions

Create a v86 filesystem from a directory:

```yaml
- name: Create v86 filesystem
  uses: arcanis/v86-fs@main
  with:
    source: ./my-filesystem
    output: ./v86-output
```

## Complete Workflow Example

Here's a complete workflow that creates a Linux filesystem for v86:

```yaml
name: Build v86 Linux Filesystem

on:
  push:
    branches: [ main ]

jobs:
  build-filesystem:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
      
      - name: Download Alpine Linux rootfs
        run: |
          wget https://dl-cdn.alpinelinux.org/alpine/v3.18/releases/x86_64/alpine-minirootfs-3.18.0-x86_64.tar.gz
          mkdir -p rootfs
          tar -xzf alpine-minirootfs-3.18.0-x86_64.tar.gz -C rootfs
      
      - name: Customize filesystem
        run: |
          # Add custom files
          echo "Welcome to v86!" > rootfs/etc/motd
          
      - name: Create v86 filesystem
        uses: arcanis/v86-fs@main
        with:
          source: ./rootfs
          output: ./v86-fs
      
      - name: Upload filesystem artifact
        uses: actions/upload-artifact@v3
        with:
          name: v86-filesystem
          path: v86-fs/v86-fs.tar.gz
      
      - name: Create release
        if: startsWith(github.ref, 'refs/tags/')
        uses: softprops/action-gh-release@v1
        with:
          files: v86-fs/v86-fs.tar.gz
```

## Using the Generated Filesystem with v86

After extracting the generated tarball, you can use it with v86:

```javascript
// Extract the tarball first
// tar -xzf v86-fs.tar.gz

var emulator = new V86({
    wasm_path: "v86.wasm",
    memory_size: 128 * 1024 * 1024,
    vga_memory_size: 8 * 1024 * 1024,
    screen_container: document.getElementById("screen_container"),
    bios: {
        url: "seabios.bin",
    },
    vga_bios: {
        url: "vgabios.bin",
    },
    bzimage: {
        url: "bzImage",
    },
    filesystem: {
        basefs: "fs.json",
        baseurl: "base/",
    },
    autostart: true,
});
```

## Building Custom Filesystems

You can create custom filesystems for specific purposes:

### Web Server Filesystem

```yaml
- name: Create web server filesystem
  run: |
    mkdir -p webfs/{var/www,etc/nginx}
    echo "<h1>Hello from v86!</h1>" > webfs/var/www/index.html
    # Add nginx config
    cp nginx.conf webfs/etc/nginx/
    
- name: Convert to v86 format
  uses: arcanis/v86-fs@main
  with:
    source: ./webfs
    output: ./v86-webfs
```

### Development Environment Filesystem

```yaml
- name: Create dev environment
  run: |
    mkdir -p devfs/{home/dev,usr/local}
    # Copy your project files
    cp -r ./myproject devfs/home/dev/
    # Add development tools
    # ... install packages, tools, etc.
    
- name: Convert to v86 format
  uses: arcanis/v86-fs@main
  with:
    source: ./devfs
    output: ./v86-devfs
```

## Testing the Filesystem Locally

Before using in production, you can test the generated filesystem:

```bash
# Extract the tarball
tar -xzf v86-fs.tar.gz

# Verify the structure
ls -l fs.json base/

# Check the JSON structure
python3 -m json.tool fs.json | less

# Verify file hashes
cd base && sha256sum *.bin
```

## Deployment to GitHub Pages

Deploy the filesystem to GitHub Pages for easy access:

```yaml
- name: Create v86 filesystem
  uses: arcanis/v86-fs@main
  with:
    source: ./rootfs
    output: ./public/9p

- name: Deploy to GitHub Pages
  uses: peaceiris/actions-gh-pages@v3
  with:
    github_token: ${{ secrets.GITHUB_TOKEN }}
    publish_dir: ./public
```

Then access it in your v86 configuration:

```javascript
filesystem: {
    basefs: "https://yourusername.github.io/yourrepo/9p/fs.json",
    baseurl: "https://yourusername.github.io/yourrepo/9p/base/",
}
```

## Performance Tips

1. **Exclude unnecessary files**: Use `.gitignore` patterns in your source directory to exclude build artifacts, logs, and temporary files
2. **Pre-compress large files**: The SHA256-based deduplication works better with smaller, uncompressed files
3. **Use CDN**: Host the `base/` directory on a CDN for better performance
4. **Cache workflow steps**: Cache the cloned v86 repository to speed up repeated builds

```yaml
- name: Cache v86 tools
  uses: actions/cache@v3
  with:
    path: /tmp/v86-tools
    key: v86-tools-${{ runner.os }}
```
