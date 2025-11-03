# Tools

This repository is a grab bag of binaries for unpacking and repacking Android OTA payloads and related filesystem images.

## Layout

- `bin/` – native executables. Add this directory to your `PATH` to call them directly.

## Binaries

| Tool | Description |
| --- | --- |
| `brotli` | Google Brotli compressor/decompressor; required when OTA payloads ship Brotli-compressed blobs. |
| `extract.erofs` | Extracts files from an EROFS image. Helpful for inspecting dynamic partitions in modern Android builds. |
| `lpunpack` | Splits a `super.img` (Logical Partitions image) into its constituent partition blobs. |
| `mkfs.erofs` | Builds an EROFS filesystem from an input directory tree. |
| `ota_extractor` | Pulls partitions and payload metadata out of `payload.bin` archives. |
| `simg2img` | Converts Android sparse images (e.g., `system.img`) into raw ext images for mounting. |
| `zstd` | Zstandard compressor/decompressor used by recent OTAs and system partitions. |

## Getting Started

Add the executables to your environment path:

```bash
export PATH="$PWD/bin:$PATH"
```

Once the path is set, call a tool with `tool-name --help` (or `-h`) to inspect the available options.

## mkota helper

The `bin/mkota` script assembles a flashable recovery package from partition images.

You must provide the device metadata (device name, firmware string, codename) using command-line flags, environment variables, or a config file:

```bash
# One-off overrides
mkota --device "Infinix GT 20 Pro" \
          --firmware "X6871-15.1.2.129(OP001PF001AZ)PORT" \
          --codename "X6871-OP" \
          /path/to/images /tmp/mkota

# Environment variables
MKOTA_DEVICE="Infinix GT 20 Pro" \
MKOTA_FIRMWARE="X6871-15.1.2.129(OP001PF001AZ)PORT" \
MKOTA_CODENAME="X6871-OP" \
mkota /path/to/images /tmp/mkota

# Config file (mkota.conf in the CWD, script directory, or ~/.config/mkota/config)
cat <<'EOF' > mkota.conf
DEVICE="Infinix GT 20 Pro"
FIRMWARE="X6871-15.1.2.129(OP001PF001AZ)PORT"
CODENAME="X6871-OP"
EOF
mkota /path/to/images /tmp/mkota
```

You can also point at an explicit configuration file with `--config /path/to/mkota.conf` or by exporting `MKOTA_CONFIG`.
