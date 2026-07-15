# Final Logo System

Pocket Mint uses one approved logo geometry: **Pocket Fold**. The mark represents a contained private workspace with a controlled folded opening. Its geometry is final and must not be redrawn, simplified, or optically altered.

The system has two approved arrangements:

1. **Primary horizontal lockup:** Pocket Fold followed by the live-text wordmark `Pocket Mint` in Inter SemiBold.
2. **Icon-only mark:** Pocket Fold used independently in compact or already-branded contexts.

No tagline or secondary lockup is part of the logo system.

## Asset Inventory

| Asset | Purpose | Color behavior |
|---|---|---|
| `pocket-fold.svg` | Master icon-only asset | `currentColor` |
| `pocket-fold-black.svg` | Fixed monochrome reproduction | Pure black `#000000` |
| `pocket-fold-white.svg` | Reversed reproduction | Pure white `#ffffff` |
| `pocket-mint-lockup.svg` | Primary horizontal lockup | Mark and live wordmark use `currentColor` |
| `favicon.svg` | Browser favicon at 16px, 32px, or `sizes="any"` | Fixed Slate `#0F172A` |

Slate and primary dark-teal icon versions are produced from `pocket-fold.svg` by setting CSS `color` to `#0F172A` or `#001414`. Separate duplicate geometry files are intentionally avoided.

## Lockup Specifications

The primary lockup is horizontal only.

- Mark box: `32 × 32px` in the master lockup.
- Wordmark: `Pocket Mint`, Inter SemiBold (`font-weight: 600`).
- Wordmark size: `24px` when the mark box is `32px`.
- Icon-to-wordmark ratio: `4:3` by nominal size.
- Optical gap: one quarter of the mark box, equal to `8px` at a 32px mark.
- Alignment: center the mark box against the wordmark line box.
- Baseline: at the 32px master size, the wordmark baseline is `24.5px` from the lockup top.
- Tagline: prohibited.
- Orientation: horizontal only; do not stack the wordmark under the mark.

For application implementation, prefer an HTML or React lockup composed of the icon component and live text. This ensures the wordmark inherits the product's loaded Inter font and avoids external-SVG font fallback.

Recommended contexts:

- Expanded desktop sidebar: 24px mark, 18px wordmark, 6px gap.
- Mobile top bar: 24px mark, 18px wordmark, 6px gap.
- Login and authentication: 32px mark, 24px wordmark, 8px gap.
- First-use identity: horizontal lockup at 32px mark size or larger.

## Clear Space

Define `X` as the mark's minimum structural section:

`X = 3/24 of the mark box = 1/8 of the mark size.`

Use `2X` as the minimum exclusion area.

- Icon-only clear space: `2X` on every side, equal to one quarter of the displayed mark size.
- Horizontal lockup clear space: `2X` around the complete lockup, calculated from the lockup's mark size.
- Icon-to-wordmark gap: `2X`.

Examples:

| Mark size | X | Minimum clear space | Lockup gap |
|---:|---:|---:|---:|
| 16px | 2px | 4px | Icon-only use |
| 20px | 2.5px | 5px | 5px |
| 24px | 3px | 6px | 6px |
| 32px | 4px | 8px | 8px |
| 48px | 6px | 12px | 12px |
| 64px | 8px | 16px | 16px |

The clear-space rule applies outside the SVG viewBox. The mark's approved internal optical padding must remain unchanged.

## Minimum Sizes

| Usage | Size | Rule |
|---|---:|---|
| Browser favicon | 16px | Minimum approved icon-only size |
| Compact navigation | 20px | Use icon-only where brand context is established |
| Standard navigation | 24px | Default application navigation size |
| Horizontal lockup | 20px mark | Minimum lockup mark size; pair with 15px wordmark and 5px gap |
| Authentication identity | 32px | Use the horizontal lockup |
| App identity | 48px | Icon-only or lockup according to available width |
| Large brand surface | 64px | Maintain the same geometry and clear-space ratio |

Do not display the icon below 16px. Use `favicon.svg` at both 16px and 32px; its `24 × 24` viewBox preserves the approved internal breathing room.

## Background Usage

| Background | Approved logo color |
|---|---|
| White `#ffffff` | Slate `#0F172A`, dark teal `#001414`, or pure black |
| Pocket Mint Surface `#f9f9f8` | Slate `#0F172A` or dark teal `#001414` |
| Slate `#0F172A` | Pure white reversed |
| Primary dark teal `#001414` | Pure white reversed |

Use only a single logo color at a time. Maintain strong contrast and keep the countershape visually distinct from the background.

## Incorrect Usage

Never:

- Alter, simplify, or redraw the path geometry.
- Rotate, skew, stretch, compress, or change the proportions.
- Change the fold opening, corner radii, optical padding, countershape, or diagonal terminals.
- Separate, extend, shorten, round, or reconnect the diagonal terminals.
- Add shadows, gradients, outlines, strokes, glows, textures, or opacity effects.
- Place the mark inside another container, badge, tile, wallet, shield, or decorative shape.
- Change the internal spacing or crop the `0 0 24 24` viewBox.
- Use arbitrary or non-semantic colors.
- Place the mark over low-contrast, visually noisy, or photographic backgrounds without a clear approved field.
- Convert the wordmark into an unapproved font or weight.
- Stack, curve, or otherwise recompose the wordmark.

## React Implementation Guidance

Application code is not modified by this package. The following examples show the recommended integration pattern.

### Icon-only component

```tsx
import type { CSSProperties } from "react";

type PocketFoldProps = {
  size?: number | string;
  className?: string;
  color?: CSSProperties["color"];
  decorative?: boolean;
  label?: string;
};

export function PocketFold({
  size = 24,
  className,
  color,
  decorative = true,
  label = "Pocket Mint",
}: PocketFoldProps) {
  return (
    <svg
      xmlns="http://www.w3.org/2000/svg"
      viewBox="0 0 24 24"
      width={size}
      height={size}
      className={className}
      style={{ color }}
      focusable="false"
      aria-hidden={decorative ? true : undefined}
      role={decorative ? undefined : "img"}
      aria-label={decorative ? undefined : label}
    >
      <path
        fill="currentColor"
        d="M5 1.75h12L13.75 5h-7A1.5 1.5 0 0 0 5.25 6.5v11.25a1.5 1.5 0 0 0 1.5 1.5h10.5a1.5 1.5 0 0 0 1.5-1.5V9l3.5-3.5V19A3.25 3.25 0 0 1 19 22.25H5A3.25 3.25 0 0 1 1.75 19V5A3.25 3.25 0 0 1 5 1.75Z"
      />
    </svg>
  );
}
```

Use `decorative={true}` when adjacent visible text already says `Pocket Mint`. Use `decorative={false}` when the icon is the only accessible brand label.

### Horizontal lockup component

```tsx
type PocketMintLockupProps = {
  markSize?: number;
  className?: string;
  labelClassName?: string;
};

export function PocketMintLockup({
  markSize = 24,
  className,
  labelClassName,
}: PocketMintLockupProps) {
  return (
    <span
      className={className}
      role="img"
      aria-label="Pocket Mint"
      style={{
        display: "inline-flex",
        alignItems: "center",
        gap: markSize / 4,
        color: "currentColor",
      }}
    >
      <PocketFold size={markSize} decorative />
      <span
        className={labelClassName}
        aria-hidden="true"
        style={{
          fontFamily: "Inter, sans-serif",
          fontSize: markSize * 0.75,
          fontWeight: 600,
          lineHeight: 1,
          letterSpacing: "-0.0125em",
          whiteSpace: "nowrap",
        }}
      >
        Pocket Mint
      </span>
    </span>
  );
}
```

The wrapper supplies the single accessible name. Both the icon and visible wordmark are hidden from the accessibility tree to prevent duplicate announcements.

Use the horizontal lockup for expanded sidebar, login, authentication, mobile top bar, and first-use contexts. Use the icon-only component for favicon, collapsed sidebar, compact navigation, and contexts where `Pocket Mint` is already clearly identified.
