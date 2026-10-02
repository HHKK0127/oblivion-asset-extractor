# Oblivion Asset Extractor

Extraction tools for The Elder Scrolls IV: Oblivion BSA v103 archives.
These scripts extract DDS textures from Oblivion BSA files and convert them to PNG.

## Requirements

- Python 3.8+
- Pillow (`pip install Pillow`) for `convert_dds_to_png.py`
- `bethesda-structs` (`pip install bethesda-structs`) for `extract_with_bs.py`

## Scripts

| Script | Purpose |
|--------|---------|
| `extract_oblivion_dds.py` | Direct BSA v103 extractor. Parses the BSA header manually and extracts target DDS files, handling zlib-compressed files with 4-byte original_size prefix. |
| `extract_menu_textures.py` | Extracts menu DDS files referenced by menu XMLs. |
| `extract_menu_textures_v2.py` | Direct BSA v103 extractor for menu DDS files with proper zlib handling. |
| `extract_with_bs.py` | Extracts menu DDS files using the `bethesda-structs` library with a raw-deflate fallback. |
| `convert_dds_to_png.py` | Converts extracted DDS files to PNG and places them into an Android assets directory. |
| `debug_bsa_bytes.py` | Debug: inspects raw bytes at file-record offsets to understand the BSA v103 compressed format. |
| `debug_bsa_layout.py` | Debug: dumps the first directory/file record layout. |
| `debug_decompress.py` | Debug: manual zlib decompression tests against a BSA file record. |

## Configuration

All scripts read paths from environment variables. If an environment variable is
not set, a default (the original developer's local path) is used.

| Environment variable | Used by | Default |
|----------------------|---------|---------|
| `OBLIVION_BSA_PATH` | All extractors and debug scripts | `D:\Cargo\Oblivion_Android\BSA\bsa_Original\Oblivion - Textures - Compressed.bsa` |
| `OBLIVION_OUTPUT_DIR` | All extractors | `C:\Users\hiroki.kogarumai\Oblivion_Android\textures_extracted` |
| `OBLIVION_DDS_SRC_DIR` | `convert_dds_to_png.py` | `C:\Users\hiroki.kogarumai\Oblivion_Android\textures_extracted\textures\menus\loading` |
| `OBLIVION_PNG_DST_DIR` | `convert_dds_to_png.py` | `C:\Users\hiroki.kogarumai\Oblivion_Android\app\src\main\assets\textures\ui` |

### Example

```powershell
$env:OBLIVION_BSA_PATH = "D:\Games\Oblivion\Data\Oblivion - Textures - Compressed.bsa"
$env:OBLIVION_OUTPUT_DIR = "C:\extracted"
python extract_oblivion_dds.py
```

## BSA v103 format notes

- Header is 36 bytes: magic (`BSA\0`), version, dir_offset, flags, dir_count, file_count, dir_names_len, file_names_len, file_flags.
- File size field packs a real size in the low 30 bits (`0x3fffffff`) and a compression flag in bit 30 (`0x40000000`).
- Compressed files are zlib streams prefixed with a 4-byte little-endian original size.
- Compression is inverted against the archive's `compressed_by_default` flag (bit 2 of archive flags).