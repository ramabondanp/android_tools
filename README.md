# Android Tools

This repository contains a curated collection of binaries and scripts for unpacking, repacking, inspecting, and modifying Android OTA payloads and filesystem images.

## Repository Layout

- `bin/` – Native executables and helper scripts. Add this directory to your `PATH` to execute them directly.

## Binaries & Scripts Overview

| Tool | Architecture | Linkage | Version / Origin | Description |
| :--- | :--- | :--- | :--- | :--- |
| `brotli` | `x86_64` | Dynamic | `1.2.0` | Google Brotli compressor/decompressor; required when OTA payloads ship Brotli-compressed blobs. |
| `extract.erofs` | `x86_64` | Static | `1.0.6` (erofs-utils `1.8.4`) | Extracts files from EROFS partition images. Helpful for inspecting dynamic partitions in modern Android builds. |
| `lpunpack` | `x86_64` | Static | AOSP | Splits a logical `super.img` (Logical Partitions image) into its constituent partition blobs. |
| `mkfs.erofs` | `x86_64` | Static | erofs-utils `1.8.4` | Builds an EROFS filesystem image from an input directory tree. |
| `mkota` | Script | N/A | Bash Script | Assembles a flashable recovery package from partition and firmware images. |
| `ota_extractor` | `x86_64` | Static | AOSP | Pulls partitions and payload metadata out of `payload.bin` archives. |
| `payload-extract` | `x86_64` | Static (PIE) | Rust | High-performance tool to extract partition images, list contents, and validate Android OTA `payload.bin` (supports local files, zip archives, and HTTP URLs). |
| `simg2img` | `x86_64` | Static | AOSP | Converts Android sparse images (e.g., `system.img`) into raw ext images for mounting. |
| `zstd` | `x86_64` | Static | `1.5.8` | Zstandard compressor/decompressor used by recent OTAs and system partitions. |
| `zstd-arm64` | `ARM64` | Static | `1.5.8` | ARM64 build of Zstandard. Bundled with `mkota` to package into update ZIPs and run directly on ARM64 Android devices during recovery installation. |

---

## Getting Started

Add the executables to your environment path:

```bash
export PATH="$PWD/bin:$PATH"
```

Once the path is set, call a tool with `tool-name --help` (or `-h`) to inspect the available options.

---

## Detailed Tool Documentation & Examples

### `payload-extract`
A multi-purpose utility written in Rust to inspect and extract OTA `payload.bin` archives. It is extremely versatile because it can fetch directly from HTTP/HTTPS URLs and work within zip files.

*   **Extract all partitions:**
    ```bash
    payload-extract extract /path/to/payload.bin
    ```
*   **List all partitions inside payload:**
    ```bash
    payload-extract list /path/to/payload.bin
    ```
*   **Show OTA metadata:**
    ```bash
    payload-extract ota-metadata /path/to/payload.bin
    ```
*   **Verify extracted partitions against hashes:**
    ```bash
    payload-extract verify /path/to/payload.bin
    ```

### `lpunpack`
Splits a logical `super.img` (dynamic partition image) into separate partitions (e.g., `system.img`, `vendor.img`, `product.img`).

*   **Usage:**
    ```bash
    lpunpack [options...] SUPER_IMAGE [OUTPUT_DIR]
    ```
*   **Examples:**
    ```bash
    # Extract all partitions to the current directory
    lpunpack super.img .
    
    # Extract only the system partition
    lpunpack -p system super.img ./output/
    ```

### `ota_extractor`
The standard AOSP tool to extract partition images from an OTA payload.

*   **Usage:**
    ```bash
    ota_extractor --payload /path/to/payload.bin --output_dir /path/to/output
    ```
*   **Incremental OTA Extraction:**
    If extracting an incremental OTA, provide the directory with the source/base partition images:
    ```bash
    ota_extractor --payload /path/to/payload.bin --input_dir /path/to/base_images --output_dir /path/to/output
    ```

### `extract.erofs` & `mkfs.erofs`
Tools for working with EROFS (Enhanced Read-Only File System), which is widely used in modern Android builds (Android 13+).

*   **Extracting an EROFS image:**
    ```bash
    extract.erofs -i system.img -x
    ```
*   **Building an EROFS image:**
    ```bash
    mkfs.erofs -d /path/to/source_directory system.img
    ```

### `simg2img`
Converts Android sparse images to raw mountable ext4 images.

*   **Usage:**
    ```bash
    simg2img <sparse_image_files> <raw_image_file>
    ```

### `zstd` & `zstd-arm64`
Zstandard (zstd) is a fast lossless compression algorithm.
*   `zstd` is for the host environment (`x86_64`).
*   `zstd-arm64` is pre-compiled for target Android devices (ARM64). It is bundled here specifically for the `mkota` script, which packages it inside update ZIPs to perform decompression on the device.

---

## `mkota` Helper Script

The `bin/mkota` script assembles a flashable recovery package from partition images.

You must provide the device metadata (device name, firmware string, codename) using command-line flags, environment variables, or a config file:

```bash
# One-off overrides
mkota --device "Infinix GT 20 Pro" \
          --firmware "X6871-15.1.2.145SP02(OP001PF001AZ)" \
          --codename "X6871" \
          /path/to/images /tmp/mkota

# Environment variables
MKOTA_DEVICE="Infinix GT 20 Pro" \
MKOTA_FIRMWARE="X6871-15.1.2.145SP02(OP001PF001AZ)" \
MKOTA_CODENAME="X6871" \
mkota /path/to/images /tmp/mkota

# Config file (mkota.conf in the CWD, script directory, or ~/.config/mkota/config)
cat <<'EOF' > mkota.conf
DEVICE="Infinix GT 20 Pro"
FIRMWARE="X6871-15.1.2.145SP02(OP001PF001AZ)"
CODENAME="X6871"
EOF
mkota /path/to/images /tmp/mkota
```

You can also point at an explicit configuration file with `--config /path/to/mkota.conf` or by exporting `MKOTA_CONFIG`.
