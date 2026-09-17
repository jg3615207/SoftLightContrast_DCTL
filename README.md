# Soft Light Contrast DCTL

A single-node version of the layer-mixer contrast technique Marcel Patillo demonstrates
in *Color Grading A Concert In DaVinci Resolve 16* (~11:00–14:30).

## The technique it replaces

    [ color, untouched ] ------\
                                >---- Layer Mixer, composite mode: Soft Light
    [ RGB Mixer: Monochrome ] -/        + curve on highlights, lift on shadows

In his words: a normal contrast control "will add saturation" and gives "muddy contrast
in the lows"; this gives "the same contrast but without the actual saturation that
normal contrast will add." On the concert footage the point was to stop the stage lights
oversaturating and pulling focus off the singer.

He then lifts highlights (curves) and shadows (lift wheel) **on the mono leg only**,
because that bends light without bending color — he wasn't sure the footage was better
than 8-bit and didn't want to separate the chroma into noise. Saturation is added back
on a *separate* node afterwards (he lands around 74).

## Why it doesn't inflate saturation

A normal contrast scales R, G and B around a pivot. Steepening the curve pushes the
channels further apart, and channel distance *is* saturation — so contrast and
saturation rise together.

Here the blend layer is a single gray channel `m = luma(R,G,B)`, and soft light applies
the **same** transfer to all three channels. Chroma ratios stay close to the original.
You get the shape without the color inflating, and the roll-off at both ends is gentle
rather than clipping.

## Install

Copy `SoftLightContrast.dctl` to:

- **Windows** `%APPDATA%\Blackmagic Design\DaVinci Resolve\Support\LUT\DCTL\`
- **macOS** `/Library/Application Support/Blackmagic Design/DaVinci Resolve/LUT/DCTL/`
- **Linux** `/home/resolve/LUT/DCTL/`

Then in Color: add a node → Effects → ResolveFX Color → **DCTL** → pick it from the
DCTL List dropdown. Right-click the LUT folder → *Refresh LUTs* if it doesn't show.

## Where to put it in the tree

Soft light hinges at 0.5 and is defined on [0,1], so it wants **display-referred,
gamma-encoded** input — Rec.709 2.4, sRGB, Gamma 2.4. Put it exactly where the
layer-mixer stack would go: **after** your CST / input LUT, never on raw log or linear.

If your mid-grey sits somewhere other than 0.5, use **Pivot** instead of moving the node.

## Controls

| Control | What it does |
|---|---|
| **Contrast Strength** | Opacity of the soft-light layer. `0` = bypass, `1` = the full technique. Above `1` stacks a genuine second layer-mixer pass, the way you'd stack nodes — not an exaggerated single blend. |
| **Pivot** | Moves the soft-light hinge off 0.5. Lower = contrast builds from further down the curve (protects highlights), higher = the opposite. |
| **Shadow Lift** | His lift-wheel move on the mono leg. Positive recovers shadow detail the contrast crushed. |
| **Highlight Recover** | His highlight curve bump on the mono leg. Positive pulls the top back. |
| **Post Saturation** | The separate saturation node that follows in his tree. Luma-weighted, `1.0` = unchanged. |
| **Mix** | Final blend against the original — equivalent to Key → Output Gain on the node. |
| **Luma Weights** | Rec709 (matches Resolve's RGB Mixer monochrome default), Rec2020, or flat Average. Average reads warmer/heavier in reds. |
| **Soft Light Model** | `Photoshop` matches Resolve's composite mode — use this for parity. `Pegtop` has no derivative kink at 0.5, slightly gentler. `Illusions` is a pure gamma remap, strongest in the deep shadows. |
| **Keep Out Of Range** | On: superwhites and sub-blacks pass through with their excursion intact instead of being clamped. Turn off if you want hard [0,1] behaviour. |

## Suggested starting point for concert / stage footage

- Contrast Strength `1.0`
- Pivot `0.45` — stage lights sit high, this keeps the hinge below them
- Shadow Lift `0.08` — his "raise the shadows to give us more information"
- Highlight Recover `0.10` — his highlight curve bump
- Post Saturation `0.95`–`1.05`, then judge it **on the face only**, not on the lights

## Parity notes

- The DCTL is a faithful port of the *math*, not a pixel-exact clone of a Resolve node
  graph. His curve and lift moves were freehand; `Shadow Lift` / `Highlight Recover` are
  smooth parametric stand-ins (`(1-m)²` and `m²` weighted), not the literal wheel response.
- `Photoshop` mode implements the W3C/Photoshop soft-light formula, which is what
  Resolve's Soft Light composite mode follows. If you want to verify against your own
  build, set Shadow Lift and Highlight Recover to `0`, Pivot to `0.5`, Saturation to
  `1.0`, and A/B the DCTL against the real layer-mixer stack.
