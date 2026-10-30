# Stillwell film LUTs

The twenty colour film looks used by the Stillwell camera app, published as open data.
They are derived from [spektrafilm](https://github.com/andreavolpato/spektrafilm) by Andrea Volpato
and shared under the same license, CC BY-SA 4.0.

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

These are per-pixel colour tables only. Spatial effects (halation, grain, coupler diffusion) are
not included.

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
| `<id>-CHANGELOG.txt` | How the table was derived, as the license asks |
| `SPEKTRAFILM_LICENSE.txt` | The license, unmodified |

The `.f16`, shaper, JSON and CHANGELOG files are byte-identical to the copies inside the app.
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
