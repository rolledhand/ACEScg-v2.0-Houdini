# ACEScg-Houdini

Houdini-tuned ACES 2.0 CG OCIO config based on the official ACEScg config, with practical defaults and file rules for real CG work.

## Status
Stable and ready to use. Built to avoid the usual Houdini OCIO friction so you don't have to. Works with external renderers, tested Arnold in Solaris (Houdini 21). Loaded this config in Nuke 17 and it transferred properly.

## Compatibility
This config is `ocio_profile_version: 2.4`, so treat it as OCIO 2.4+ only: [OCIO 2.4 shipped in September 2024](https://opencolorio.readthedocs.io/en/latest/releases/ocio_2_4.html) and is part of the [VFX Reference Platform CY2025](https://vfxplatform.com/), and [Houdini 21 ships with OpenColorIO 2.4.1](https://www.sidefx.com/docs/houdini/news/21/platforms.html) ([SideFX also lists 2.4.1 in its third-party libraries](https://www.sidefx.com/docs/houdini/licenses/)), so Houdini 21 is the intended host. [Nuke 17 ships with OCIO 2.4.2](https://learn.foundry.com/nuke/17.0v1-beta4/content/release_notes/nuke_17.0.html) and is also a confirmed target.

[Resolve/Fusion 19 only documents OpenColorIO 2.3 support](https://documents.blackmagicdesign.com/SupportNotes/DaVinci_Resolve_19_New_Features_Guide.pdf?_v=1712905211000), while [Resolve 20 separately adds ACES 2.0 support](https://documents.blackmagicdesign.com/SupportNotes/DaVinci_Resolve_20_New_Features_Guide.pdf?_v=1745391610000) in its own color-management operations, whether to trust a non-OCIO implementation is your call. This means OCIO 2.4 configs should be treated as unsupported there unless your exact build documents 2.4+.

Do not assume a host-specific ACES/DRCM path is a clean workaround for standard OCIO interchange: it can diverge, so validate end-to-end instead of trusting the label, as noted on [ACESCentral](https://community.acescentral.com/t/resolve-color-management-drt-in-ocio/5289).

After Effects 26 is not a supported host. Adobe doesn't release documentation about their OCIO version, but I quickly tested it myself. Its current OCIO implementation does not load `ocio_profile_version: 2.4`.

## Software Support / Updates
For app support and current release status, start here:

- [Nuke 17 release notes](https://learn.foundry.com/nuke/17.0v1-beta4/content/release_notes/nuke_17.0.html)
- [Nuke product page](https://www.foundry.com/products/nuke/features)
- [DaVinci Resolve](https://www.blackmagicdesign.com/products/davinciresolve)
- [Fusion](https://www.blackmagicdesign.com/products/fusion)

For current release status and OCIO/ACES updates, check the links above.

## Quick Start
Create or edit `ocio.json` in your Houdini `packages` folder and replace the path with your local path of the config:

```json
{
  "enable": true,
  "show": true,
  "load_package_once": true,
  "env": [
    { "OCIO": "${OCIO-/path/to/ACEScg-v2.0-Houdini/ACEScg-v2.0_ocio-v2.4.ocio}" }
  ]
}
```

Restart Houdini after adding or changing the package file, since OCIO is read at startup.

With this config active, `ACEScg` is the intended default working space for CG content.

Houdini’s OCIO editor is unreliable in practice for config authoring/persistence (this isn’t specific to this config). If you want changes, edit the .ocio file directly and restart Houdini.

Solaris Render Gallery has its own caching/consistency quirks; don’t use it as your only ground truth when validating color.

## What This Changes
The stock ACES CG config is already the right kind of base for Houdini. The real problem was not the working space, it was default file assignment. By default, unmatched files fell back to `ACES2065-1`, which is a poor day-to-day assumption for CG work in Houdini.

`./ACEScg-v2.0_ocio-v2.4.ocio` keeps the ACES 2 structure intact, but makes the defaults practical so standard textures, EXRs, and loosely tagged files behave like a sane Houdini pipeline instead of an interchange/archive config.

| Setting | Stock ACES CG config | Houdini-tuned config |
| --- | --- | --- |
| `default` | not set | `ACEScg` |
| `reference` | not set | `ACEScg` |
| `scene_linear` | `ACEScg` | `ACEScg` |
| `color_timing` | `ACEScct` | `ACEScct` |
| `compositing_log` | `ACEScct` | `ACEScct` |
| `matte_paint` | `ACEScct` | `ACEScct` |
| `color_picking` | `sRGB Encoded Rec.709 (sRGB)` | `sRGB Encoded Rec.709 (sRGB)` |
| `texture_paint` | `sRGB Encoded Rec.709 (sRGB)` | `sRGB Encoded Rec.709 (sRGB)` |
| `Default` file rule | `ACES2065-1` | `ACEScg` |

File rules in `./ACEScg-v2.0_ocio-v2.4.ocio`:

- `*srgb_tx*` -> `sRGB Encoded Rec.709 (sRGB)`
- `*srgb_texture*` -> `sRGB Encoded Rec.709 (sRGB)`
- `*ACEScg*` -> `ACEScg`
- `__usdz_jpg` -> `sRGB Encoded Rec.709 (sRGB)`
- `__usdz_jpeg` -> `sRGB Encoded Rec.709 (sRGB)`
- `__usdz_png` -> `sRGB Encoded Rec.709 (sRGB)`
- `__usdz_tif` -> `sRGB Encoded Rec.709 (sRGB)`
- `__usdz_tiff` -> `sRGB Encoded Rec.709 (sRGB)`
- `__usdz_exr` -> `ACEScg`
- `.jpg` -> `sRGB Encoded Rec.709 (sRGB)`
- `.jpeg` -> `sRGB Encoded Rec.709 (sRGB)`
- `.png` -> `sRGB Encoded Rec.709 (sRGB)`
- `.tif` -> `sRGB Encoded Rec.709 (sRGB)`
- `.tiff` -> `sRGB Encoded Rec.709 (sRGB)`
- `.exr` -> `ACEScg`
- fallback `Default` -> `ACEScg`

Tag rules take precedence over extension rules.
USDZ archive-internal file paths are matched by explicit regex rules before the normal extension rules.

## Troubleshooting
### File rules are a safety net (not a replacement for correct MaterialX assignment)
The file rules in this config are a **pipeline fallback** for loosely named assets. OCIO file rules are evaluated **top-down**, and the first match wins.  
See [OCIO file rules documentation](https://opencolorio.readthedocs.io/en/latest/guides/authoring/authoring.html#file-rules)

Houdini/USD can expose files inside `.usdz` archives as `.usdz[texture.ext]` paths. That is why this config adds explicit regex rules for USDZ payloads, because normal extension rules are not enough on their own for those archive-internal references. See the [SideFX USD Zip render node](https://www.sidefx.com/docs/houdini/nodes/out/usdzip.html).

Do **not** treat extension rules (e.g. `.png`/`.tif` → sRGB) as a license to stop assigning the correct interpretation on MaterialX nodes:
- **Color textures** (albedo/baseColor/emission) are typically sRGB.
- **Data textures** (normal, roughness, metallic, AO, displacement, masks/IDs) must be **Raw/Data** (no color transform).

SideFX explicitly notes that when reading **normal maps** with MaterialX, you should set the **Signature to Vector3** to make sure a color space is not applied:  
[SideFX MaterialX documentation](https://www.sidefx.com/docs/houdini/solaris/materialx.html)

### Wrong-Looking Textures
If a texture looks washed out, oversaturated, or double-transformed, check whether it is already display-referred, whether the filename includes `srgb_tx`, `srgb_texture`, or `ACEScg`, and whether the extension matches the intended automatic rule.

- `albedo.png` -> `sRGB Encoded Rec.709 (sRGB)` (extension rule)
- `albedo_srgb_tx.png` -> `sRGB Encoded Rec.709 (sRGB)` (tag rule, takes precedence)
- `lighting.exr` -> `ACEScg` (extension rule)
- `lighting_ACEScg.exr` -> `ACEScg` (tag rule)

### Validating With `ociocheck`
```bash
ociocheck --iconfig ACEScg-v2.0_ocio-v2.4.ocio
```

If you need to inspect image files directly, [OpenImageIO documentation](https://openimageio.readthedocs.io/en/stable/) is a good reference, and tools like `iinfo` and `oiiotool` are useful for checking metadata, channels, and file properties.

## Validation
`ociocheck` passes for this config. Unlike the stock `Default -> ACES2065-1` behavior, this tuned version now falls back to `ACEScg`, which is the practical working assumption for Houdini CG use.

## Official / Reference Resources
For the spec, the host app, and current software support, start here:

- [ASF OCIO config Github page](https://github.com/AcademySoftwareFoundation/OpenColorIO-Config-ACES/releases)
- [ACES documentation](https://docs.acescentral.com)
- [OpenColorIO documentation](https://opencolorio.readthedocs.io/en/stable/)

## Resources for the curious ones
For deeper reading on color science and pipeline decisions:

- [Liam Collod's Picture Lab](https://liamcollod.xyz/picture-lab-lxm/lxmpicturelab.al.sorted-color.bg-black)
- [Chris Brejon's articles](https://chrisbrejon.com/articles/)
- [Chris Brejon: OCIO display transforms and misconceptions (mea culpa)](https://chrisbrejon.com/articles/ocio-display-transforms-and-misconceptions/#mea-culpa)
- [Chris Brejon: What makes a good picture formation?](https://chrisbrejon.com/articles/what-makes-a-good-picture-formation/)
- [ACESCentral: Per-channel display transform with wider rendering gamut](https://community.acescentral.com/t/per-channel-display-transform-with-wider-rendering-gamut/3768/10)

## Related Projects
[chrisbrejon/ARRI-REVEAL-OCIO-Config](https://github.com/chrisbrejon/ARRI-REVEAL-OCIO-Config) is a rather universal ARRI Reveal community reference.
[ARRI-Houdini](https://github.com/rolledhand/ARRI-Houdini) if you want the ARRI Reveal config on ACES 1.3 variant. Tweaked for Houdini by me.

This repo focuses on Houdini-friendly defaults, file rules, and predictable CG working space behavior instead. `ACEScg` as working space is a pipeline sanity choice. Open to feedback and solutions.

## Tweaking tips for the brave ones
The ACES 2.0 base is cleaner than the ARRI config which runs on ACES 1.3, but the same rule still applies: don't trust a pipeline until you validate the whole chain.

Test transfers between input texture encoding to scene-linear (Solaris/MaterialX), scene-linear to Hydra delegate/viewport, Arnold `maketx`/`.tx` generation to MaterialX reads (`.tx` + colorspace), husk/kick EXR to comp/viewers, scene-linear to display transform, and display transform to final deliverables, then lock it in your comp/grading software of choice.

The exact edge cases can still differ by render engine, delegate, and host app build. If anything looks suspicious, validate the chain end-to-end instead of assuming the label is right. Terminal with `iinfo`, `oiiotool`, and `ociocheck` becomes your best friend.
