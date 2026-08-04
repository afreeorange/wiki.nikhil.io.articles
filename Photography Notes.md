Assorted notes from me trying to teach myself Photography.

## The Triangle

It's a balance between Aperture, Shutter Speed, and ISO.

## Metering

Your camera will try and advise you. People used to use light meters for this. It will say its a good exposure if it is "18% Gray" which is what light meters were historically calibrated to (the caps on Kodak film and the insides of dedicated camera bags are this gray too.)

## The Histogram

- Luminance value on x-axis and pixel count on y-axis.
- Leftmost edge is Luminance of zero, Rightmost is Luminance of one. Any pixels touching these edges are lost information, they're just crushed out of existence. You can use Zebra stripes to tell you which parts of your image are/will be "blown out" in either direction.
- You section it into, say, five equal portions. From L-R, you are looking at _blacks_, _shadows_, _midtones_, _highlights_, and _whites_.

Usually you want a histogram that looks like a mountain or is kinda balanced out but depends on what you're trying to do, the look you're going for.

## On Aperture

Measured by f-stop. "f/2" means you take the focal length and divide it by two.

"[f/8 and Be There.](https://www.youtube.com/watch?v=DQcFQ-2dtB0)." Like _be there_. Your camera primarily exists to take pictures of things and events.

## On ISO

[See this video](https://www.youtube.com/watch?v=ZWSvHBG7X0w). TLDR: This is tricky so let the camera figure it out.

## Tripods

- Cheap, Stable, Light —- pick two. But don't be cheap. You'll change cameras before tripods.
- [Read this](/assets/tripods-bythom.html) thoroughly.
- Components are: Legs, Head, Plate, Bag
  - Each component may be purchased separately.
  - Buying a head doesn't necessarily include a plate.

## Lenses

There are Prime lenses (fixed focal length) and Zoom lenses (can be wide, normal, and/or telephoto lenses).

Prime lenses teach you a lot about photography because they make your lazy ass _move_ and _explore_ a scene: distances and angles.

## Presets and LUTs

- [Presetmator](https://presetmator.com/) -- Photomator/Pixelmator
- [Bodega Supply](https://bodega.supply/collection/photo-presets) -- Photomator/Pixelmator
- [FujifilmCameraProfiles](https://github.com/abpy/FujifilmCameraProfiles)
- [Export LUT](https://johnrellis.com/lightroom/exportlut.htm) Utility

### Hald CLUTs

[These are great](https://github.com/cedeber/hald-clut)!. This gives you a bunch of PNGs that you can "infer LUTs" from. Quick Python script that will do this is at the end of this page.

Now the LUT grid size with the PNGs is 144 x 144 x 144 = 2,985,984 which makes the sizes of the `.cube` files explode. That script will downsample to 32 x 32 x 32.

[I've saved the generated `.cube` files here](https://public.nikhil.io/hald-clut-cubes.tgz).

## Other

- [What This NYT/Getty/AP Photojournalist Has Learned After More Than 25 Years in the Field](https://www.youtube.com/watch?v=Ky78k8j2-zc)
- [How Metering Works](https://www.youtube.com/watch?v=qYGc7QPU3tI) -- A must-watch

### Infer LUT from PNG Script

```python
#!/usr/bin/env python3
"""Infer a 3D color LUT from images and export as .cube file.

Modes:
  hald   — Extract LUT from a processed Hald CLUT image.
  pair   — Approximate LUT from a before/after image pair.
  generate-hald — Generate a neutral Hald CLUT image to use as a reference.

Usage:
  python infer_lut.py hald processed_hald.png -o my.cube
  python infer_lut.py pair original.png graded.png -o my.cube
  python infer_lut.py generate-hald -s 8 -o identity_hald.png
"""

import argparse
import numpy as np
from pathlib import Path
from PIL import Image


# ---------------------------------------------------------------------------
# Hald CLUT helpers
# ---------------------------------------------------------------------------


def hald_level_from_image(img: Image.Image) -> int:
    """Determine the Hald level from image dimensions.

    A Hald CLUT of level N has dimensions (N^3) x (N^3).
    The resulting 3D LUT has N^2 entries per axis.
    Common levels: 4 → 64x64 image, 16-entry LUT
                   8 → 512x512 image, 64-entry LUT
                  12 → 1728x1728 image, 144-entry LUT
    """
    w, h = img.size
    if w != h:
        raise ValueError(f"Hald CLUT must be square, got {w}x{h}")
    # w = level^3
    level = round(w ** (1 / 3))
    if level**3 != w:
        raise ValueError(
            f"Image size {w}x{h} is not a valid Hald CLUT dimension (need N^3 x N^3)"
        )
    return level


def generate_hald(level: int) -> Image.Image:
    """Generate a neutral (identity) Hald CLUT image."""
    lut_size = level * level  # entries per axis in the 3D LUT
    dim = level**3  # pixel width/height of the Hald image
    pixels = np.zeros((dim * dim, 3), dtype=np.uint8)

    for i in range(dim * dim):
        r = i % lut_size
        g = (i // lut_size) % lut_size
        b = i // (lut_size * lut_size)
        pixels[i] = [
            round(r / (lut_size - 1) * 255),
            round(g / (lut_size - 1) * 255),
            round(b / (lut_size - 1) * 255),
        ]

    return Image.fromarray(pixels.reshape(dim, dim, 3))


def lut_from_hald(img: Image.Image) -> tuple[int, np.ndarray]:
    """Extract a 3D LUT from a processed Hald CLUT image.

    Returns (lut_size, lut) where lut has shape (lut_size, lut_size, lut_size, 3)
    with float values in [0, 1].
    """
    level = hald_level_from_image(img)
    lut_size = level * level
    dim = level**3

    pixels = np.array(img.convert("RGB"), dtype=np.float64).reshape(-1, 3) / 255.0
    expected = dim * dim
    if len(pixels) != expected:
        raise ValueError(f"Expected {expected} pixels, got {len(pixels)}")

    lut = np.zeros((lut_size, lut_size, lut_size, 3), dtype=np.float64)
    for i in range(expected):
        r = i % lut_size
        g = (i // lut_size) % lut_size
        b = i // (lut_size * lut_size)
        lut[b, g, r] = pixels[i]

    return lut_size, lut


def resample_lut(lut: np.ndarray, src_size: int, dst_size: int) -> np.ndarray:
    """Resample a 3D LUT to a different grid size via trilinear interpolation."""
    from scipy.interpolate import RegularGridInterpolator

    axis = np.linspace(0, 1, src_size)
    channels = []
    for ch in range(3):
        interp = RegularGridInterpolator(
            (axis, axis, axis), lut[:, :, :, ch], method="linear"
        )
        dst_axis = np.linspace(0, 1, dst_size)
        grid = np.meshgrid(dst_axis, dst_axis, dst_axis, indexing="ij")
        pts = np.stack(grid, axis=-1).reshape(-1, 3)
        channels.append(interp(pts).reshape(dst_size, dst_size, dst_size))

    return np.clip(np.stack(channels, axis=-1), 0, 1)


# ---------------------------------------------------------------------------
# Before/after pair → LUT approximation
# ---------------------------------------------------------------------------


def lut_from_pair(
    original: Image.Image,
    graded: Image.Image,
    lut_size: int = 33,
) -> tuple[int, np.ndarray]:
    """Approximate a 3D LUT by comparing original ↔ graded pixel pairs.

    Bins every pixel by its original RGB value into a 3D grid, averages the
    corresponding graded values per bin. Fills empty bins via nearest-neighbor
    interpolation.
    """
    if original.size != graded.size:
        # resize graded to match original
        graded = graded.resize(original.size, Image.LANCZOS)

    orig_px = np.array(original.convert("RGB"), dtype=np.float64).reshape(-1, 3)
    grad_px = np.array(graded.convert("RGB"), dtype=np.float64).reshape(-1, 3)

    # Quantize original pixels into LUT bins
    bin_indices = np.clip(
        (orig_px / 255.0 * (lut_size - 1) + 0.5).astype(int), 0, lut_size - 1
    )

    # Accumulate
    lut_sum = np.zeros((lut_size, lut_size, lut_size, 3), dtype=np.float64)
    lut_count = np.zeros((lut_size, lut_size, lut_size), dtype=np.float64)

    for idx in range(len(orig_px)):
        r, g, b = bin_indices[idx]
        lut_sum[b, g, r] += grad_px[idx]
        lut_count[b, g, r] += 1

    # Average filled bins
    mask = lut_count > 0
    lut = np.zeros_like(lut_sum)
    lut[mask] = lut_sum[mask] / lut_count[mask, np.newaxis] / 255.0

    # Fill empty bins via nearest-neighbor from filled bins
    empty = ~mask
    n_empty = empty.sum()
    if n_empty > 0:
        from scipy.ndimage import distance_transform_edt

        # distance_transform_edt returns distances and indices of nearest filled cell
        _, nearest_idx = distance_transform_edt(
            empty, return_distances=True, return_indices=True
        )
        for b_i in range(lut_size):
            for g_i in range(lut_size):
                for r_i in range(lut_size):
                    if empty[b_i, g_i, r_i]:
                        nb, ng, nr = nearest_idx[:, b_i, g_i, r_i]
                        lut[b_i, g_i, r_i] = lut[nb, ng, nr]

    return lut_size, lut


# ---------------------------------------------------------------------------
# .cube export
# ---------------------------------------------------------------------------


def write_cube(path: str, lut_size: int, lut: np.ndarray, title: str = "Inferred LUT"):
    """Write a 3D LUT in Adobe .cube format.

    lut shape: (lut_size, lut_size, lut_size, 3) with values in [0, 1].
    Iteration order: R fastest, then G, then B (standard .cube convention).
    """
    with open(path, "w") as f:
        f.write(f'TITLE "{title}"\n')
        f.write(f"LUT_3D_SIZE {lut_size}\n")
        f.write(f"DOMAIN_MIN 0.0 0.0 0.0\n")
        f.write(f"DOMAIN_MAX 1.0 1.0 1.0\n\n")

        for b in range(lut_size):
            for g in range(lut_size):
                for r in range(lut_size):
                    rv, gv, bv = lut[b, g, r]
                    f.write(f"{rv:.6f} {gv:.6f} {bv:.6f}\n")


# ---------------------------------------------------------------------------
# CLI
# ---------------------------------------------------------------------------


def main():
    parser = argparse.ArgumentParser(
        description="Infer a 3D color LUT from images.",
        formatter_class=argparse.RawDescriptionHelpFormatter,
        epilog=__doc__,
    )
    sub = parser.add_subparsers(dest="command", required=True)

    # -- hald --
    p_hald = sub.add_parser("hald", help="Extract LUT from a processed Hald CLUT image")
    p_hald.add_argument("image", help="Processed Hald CLUT image (PNG, TIFF, etc.)")
    p_hald.add_argument(
        "-o", "--output", default="output.cube", help="Output .cube file"
    )
    p_hald.add_argument(
        "-s",
        "--size",
        type=int,
        default=35,
        help="Output LUT grid size per axis (default 35)",
    )

    # -- pair --
    p_pair = sub.add_parser("pair", help="Approximate LUT from original + graded pair")
    p_pair.add_argument("original", help="Original (ungraded) image")
    p_pair.add_argument("graded", help="Graded image")
    p_pair.add_argument(
        "-o", "--output", default="output.cube", help="Output .cube file"
    )
    p_pair.add_argument(
        "-s",
        "--size",
        type=int,
        default=33,
        help="LUT grid size per axis (default 33, industry standard)",
    )

    # -- generate-hald --
    p_gen = sub.add_parser("generate-hald", help="Generate a neutral Hald CLUT image")
    p_gen.add_argument(
        "-s",
        "--level",
        type=int,
        default=8,
        help="Hald level (default 8 → 512x512 image, 64-entry LUT)",
    )
    p_gen.add_argument(
        "-o", "--output", default="identity_hald.png", help="Output image path"
    )

    args = parser.parse_args()

    if args.command == "generate-hald":
        print(f"Generating identity Hald CLUT (level {args.level})...")
        img = generate_hald(args.level)
        img.save(args.output)
        lut_size = args.level**2
        print(f"Saved {img.size[0]}x{img.size[1]} image → {args.output}")
        print(
            f"Resulting LUT will have {lut_size} entries per axis ({lut_size}^3 = {lut_size**3} total)"
        )

    elif args.command == "hald":
        print(f"Reading Hald CLUT: {args.image}")
        img = Image.open(args.image)
        lut_size, lut = lut_from_hald(img)
        if args.size != lut_size:
            print(f"Resampling {lut_size}³ → {args.size}³...")
            lut = resample_lut(lut, lut_size, args.size)
            lut_size = args.size
        write_cube(
            args.output,
            lut_size,
            lut,
            title=f"Hald-inferred from {Path(args.image).name}",
        )
        print(f"Extracted {lut_size}x{lut_size}x{lut_size} LUT → {args.output}")

    elif args.command == "pair":
        print(f"Original: {args.original}")
        print(f"Graded:   {args.graded}")
        orig = Image.open(args.original)
        grad = Image.open(args.graded)
        lut_size, lut = lut_from_pair(orig, grad, args.size)
        write_cube(
            args.output,
            lut_size,
            lut,
            title=f"Pair-inferred from {Path(args.original).name} → {Path(args.graded).name}",
        )
        print(f"Approximated {lut_size}x{lut_size}x{lut_size} LUT → {args.output}")

    print("Done.")


if __name__ == "__main__":
    main()
```

