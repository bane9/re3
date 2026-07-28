# Vendored librw

This directory used to be a git submodule of https://github.com/aap/librw.
It is now bundled directly in this repository so the renderer builds without
any submodule setup, and so our fixes travel with the code.

**Upstream revision this copy is based on:** `5501c4fdc7425ff926be59369a13593bb6c81b54`

## Local changes vs upstream

Both fix heap corruption when loading textures from modded TXD files.

### 1. `src/gl/gl3raster.cpp` — correct size for DXT levels (`getLevelSize`)

Compressed (DXT) mip levels were sized as `stride * height`, which is only
correct when both dimensions are multiples of 4. For a 70x70 DXT1 texture that
yields 2450 bytes, while the real size is `ceil(70/4) * ceil(70/4) * 8 = 2592`
— so `flipDXT()` wrote 142 bytes past the end of the allocation and corrupted
the heap. The size is now computed from 4x4 blocks:

    ((w+3)/4) * ((h+3)/4) * blockSize      // blockSize: 8 for DXT1, 16 for DXT3/DXT5

### 2. `src/gl/gl3raster.cpp` and `src/d3d/d3d8.cpp` — clamp texture level reads

`readNativeTexture()` read the per-level byte count out of the TXD file and
passed it straight to `stream->read8(data, size)`, with no check that it fits
the buffer returned by `raster->lock()`. A TXD declaring a level larger than
the raster overflowed the heap. Reads are now clamped to the level size and the
remainder is skipped, and the GL3 reader also skips levels beyond
`getNumLevels()` (the D3D8 reader already did).

## Updating librw

Diff this tree against the upstream revision above, pull upstream changes, then
re-apply the two fixes (or upstream them).
