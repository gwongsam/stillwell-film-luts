# Stillwell film LUTs

The twenty colour film looks used by the Stillwell camera app, published as open data.
They are derived from [spektrafilm](https://github.com/andreavolpato/spektrafilm) by Andrea Volpato
and shared under the same license, CC BY-SA 4.0.

The collection now also includes a separate grain table for each stock.

```
Derived from spektrafilm by Andrea Volpato
https://github.com/andreavolpato/spektrafilm
Licensed CC BY-SA 4.0
Modified by gwongsam (Stillwell).
```

Keep this notice, the license file and each stock's CHANGELOG with every copy or derivative.

## What was modified

The original spektrafilm profiles were not modified. For each film (and, for negatives, the print
material it is printed on), a small parametric model was built from those profiles, fitted against
reference renderings of that pairing, and baked into a 65³ lookup table. Each stock's
`*-CHANGELOG.txt` records the exact profiles, upstream commit (3bb2c2d), fit and bake, and the
measured baking error.

The colour tables are per-pixel; grain is supplied as separate data tables. Halation and coupler
diffusion are not included.

The grain tables use spektrafilm's light-sensitive layer and sublayer data
(`density_curves_layers`) to derive grain strength for each layer and sublayer. Five parameters
per stock (three channel strengths, a spatial cell size and a variance floor), plus one shared
sublayer amplitude ratio, were fitted to grain statistics on flat patches in reference
renderings: Lab standard deviations and pixel autocorrelation. The original profiles remain
unmodified; the resulting grain tables are shared under the same CC BY-SA 4.0 license.

## Stocks

Negatives are baked together with a print material, so each table is one film × print pairing.
Slides are viewed directly.

| Name | Folder | Film profile | Print profile |
|---|---|---|---|
| Cine 5207 | `luts/Cine5207` | `kodak_vision3_250d` | `kodak_2383` |
| Cine 5203 | `luts/Cine5203` | `kodak_vision3_50d` | `kodak_2383` |
| Cine 5213 | `luts/Cine5213` | `kodak_vision3_200t` | `kodak_2383` |
| Cine 5219 | `luts/Cine5219` | `kodak_vision3_500t` | `kodak_2383` |
| Verita 200D | `luts/Verita200D` | `kodak_verita_200d` | `kodak_2383` |
| Gold 200 | `luts/Gold200` | `kodak_gold_200` | `kodak_portra_endura` |
| Ultramax 400 | `luts/Ultramax400` | `kodak_ultramax_400` | `kodak_portra_endura` |
| Portra 160 | `luts/Portra160` | `kodak_portra_160` | `kodak_portra_endura` |
| Portra 400 | `luts/Portra400` | `kodak_portra_400` | `kodak_portra_endura` |
| Portra 800 | `luts/Portra800` | `kodak_portra_800` | `kodak_portra_endura` |
| Portra 800 +1 | `luts/Portra800Push1` | `kodak_portra_800_push1` | `kodak_portra_endura` |
| Portra 800 +2 | `luts/Portra800Push2` | `kodak_portra_800_push2` | `kodak_portra_endura` |
| Ektar 100 | `luts/Ektar100` | `kodak_ektar_100` | `kodak_portra_endura` |
| C200 | `luts/C200` | `fujifilm_c200` | `fujifilm_crystal_archive_typeii` |
| X-Tra 400 | `luts/XTra400` | `fujifilm_xtra_400` | `fujifilm_crystal_archive_typeii` |
| Pro 400H | `luts/Pro400H` | `fujifilm_pro_400h` | `fujifilm_crystal_archive_typeii` |
| Kodachrome 64 | `luts/Chroma64` | `kodak_kodachrome_64` | none (slide, viewed directly) |
| Provia 100F | `luts/Provia100F` | `fujifilm_provia_100f` | none (slide, viewed directly) |
| Velvia 100 | `luts/Velvia100` | `fujifilm_velvia_100` | none (slide, viewed directly) |
| Ektachrome 100 | `luts/Ektachrome100` | `kodak_ektachrome_100` | none (slide, viewed directly) |

## Files in each folder

| File | What it is |
|---|---|
| `<id>-cube65.f16` | The table exactly as the app ships it: 65³ × RGB, little-endian IEEE half floats |
| `<id>-cube65.cube` | The same table as text (float32 values, nine significant digits) |
| `<id>-shaper1024.f32` | 1024-point input shaper, little-endian float32 |
| `<id>-lut.json` | Full encoding, provenance and error budgets |
| `<id>-grain.f32` | 256 grain-strength samples and exposure-rewrite matrices, plus cell size and version; 3076 little-endian float32 values |
| `<id>-grain.json` | Grain-table layout, provenance, fitted parameters, measured acceptance statistics and exceptions |
| `<id>-CHANGELOG.txt` | How the table was derived, as the license asks |
| `SPEKTRAFILM_LICENSE.txt` | The license, unmodified |

The `.f16`, shaper, grain, JSON and CHANGELOG files are byte-identical to the copies inside the app.
`lut.json` also lists a binary `.f32` table; it is left out here because the `.cube` carries the
same float32 values.

## How to apply a table

The input is **scene-linear ProPhoto RGB (ROMM primaries, D50)** with middle grey at 0.18.
For each channel value `v`:

1. Log encode: `e = clamp((log2(max(v, 0.18 · 2⁻⁹) / 0.18) + 9) / 17, 0, 1)`
   (−9 to +8 EV around middle grey).
2. Shape: `s = P(e)`, where `P` is the 1024-point shaper sampled at `e = k / 1023`, interpolated
   linearly.
3. Look up `(s_R, s_G, s_B)` in the 65³ table with **tetrahedral** interpolation (the error budgets
   were measured that way). The table is stored with R varying fastest, then G, then B, which is the
   usual `.cube` order. The `.cube` file's `DOMAIN` refers to the shaper output, not to `v`.

The output is **linear ProPhoto RGB (D50), display-referred and unclipped**: the film and print
response is already in it, so convert it to your display space (for example sRGB or Rec. 709)
without adding another tone curve. Because of the shaper, the `.cube` file alone is not a drop-in
LUT for grading software; the shaper step has to be applied first.

## How to apply the grain table

Apply grain to the same log-encoded input `eₙ` as step 1 above, **before the shaper**.
Each `-grain.f32` contains 3076 little-endian IEEE float32 values (12,304 bytes). Indices below
are zero-based and ranges exclude their upper endpoint. The 256 nodes are `eₙ,i = i / 255`.

| Float indices | Contents |
|---|---|
| `[0, 768)` | Grain strength `G_c(eₙ,i)` in density · µm, at `i · 3 + c`, with c = R, G, B |
| `[768, 3072)` | 256 row-major 3 × 3 matrices `K_i`, at `768 + i · 9 + row · 3 + col`; row is the output exposure channel, col is the density channel |
| `3072` | Spatial cell size `cell_um`, in µm |
| `[3073, 3076)` | Format version `3`, followed by two zero padding values |

Interpolate `G_R`, `G_G` and `G_B` linearly at their **own channel's** `eₙ` value. Interpolate all
nine entries of `K` at the single mean `ē = (eₙ,R + eₙ,G + eₙ,B) / 3`. Clamp interpolation
coordinates to the endpoints. `K` maps density perturbations back to log exposure; it is the
regularized inverse of the model's density Jacobian, with `λ = 0.02 · ‖J‖_F` and
`K = (JᵀJ + λ²I)⁻¹Jᵀ` (zero where J is exactly zero).

Generate one independent, zero-mean random field `U_c` per channel, with low-frequency power
spectral density `W0 = 1 µm²` and spatial correlation scale `cell_um`. Map the image onto a
135 film frame: for long and short image dimensions `L` and `S` in pixels,

```text
long_side_mm = min(36, 24 · L / S)
p = L / (1000 · long_side_mm)            pixels per µm
cell_pixels = cell_um · p
```

Normalize the fields in film-area units and integrate them over the pixel footprint. In
particular, preserve `W0` when a cell is smaller than a pixel; unit-variance pixel noise at every
resolution would change the grain's physical strength. `cell_um` sets the spatial scale, while
the sampled `G` already contains the fitted channel strengths, sublayer ratio and variance floor.
Do not multiply those parameters into `G` again.

For each pixel, use componentwise multiplication for `G · U`, matrix multiplication for
`K · δD`, and componentwise clamps:

```text
δD = m · G · U
δeₙ = clamp(K(ē) · δD, −1/17, +1/17)
eₙ′ = clamp(eₙ + δeₙ, 0, 1)
```

Then apply the usual shaper to `eₙ′` and look up the colour table. `m = 1` is that stock's own
calibrated grain; `m = 0` leaves the original colour-table input unchanged. For video, generate
an independent field for each frame.

The grain JSON's `acceptance` values are measured statistics, not limits: `B_median` and `B_max`
are the median and maximum absolute natural-log ratios of Lab grain standard deviations on
visible neutral patches; `Bprime_max_ratio` is the largest absolute error divided by its allowed
error, `max([0.25, 0.5, 0.5], 0.35 · reference_sigma)` for L*, a*, b*. `C_L` and `C_C` are the
median absolute natural-log ratios of luminance and chroma grain standard deviations on colour
patches. Neutral patches enter B when reference `sigma_L` is at least 0.25; B′ includes all
neutral patches. For the four slide stocks, neutral highlights at +3 EV and above are excluded
from B and B′: their lower highlight grain is an intentional model choice. Each JSON records any
accepted exception separately.

## Accuracy

Against the fitted model, every table stays within these CIEDE2000 limits: median 0.2, p99 0.5,
p99.9 1.0, colour chart 1.0, grey axis 0.3. The actual values are in each `lut.json`
(`error_on_frozen_model`).

## The author's requests

Beyond the legal terms, the spektrafilm author asks (see `SPEKTRAFILM_LICENSE.txt`) that the LUTs
not be repackaged and sold as a paid LUT pack, that they not be used to train commercial AI models,
and that the spektrafilm name not be used in product branding. Please respect those too.

## Trademarks

Kodak, Fujifilm and the film product names are trademarks of their respective owners. They are
used only to identify the stocks these tables emulate. This project is not affiliated with or
endorsed by them, or by spektrafilm.

---

## 中文说明

这里是「叠光（Stillwell）」相机 App 成像层用的 20 款彩色胶片 LUT。它们派生自 Andrea Volpato 的
[spektrafilm](https://github.com/andreavolpato/spektrafilm) 数据，按同一许可 CC BY-SA 4.0 公开，
由 gwongsam（Stillwell）修改。原始 profile 未作修改；负片是「胶片 × 相纸」整条链路烘进一张表，
反转片直接看片。用法、文件格式和误差见上文；转载或再派生时请保留署名、许可文件和各款 CHANGELOG。
