# Implementation Plan: Fix Mobile GameTable Live Clock Visibility

## Cause Analysis
The live clock was positioned using negative top offset (`-top-9`) outside the bottom HUD container. On mobile viewports, the outer GameTable container has `overflow-hidden`, which clips and hides elements positioned outside their parent container boundaries.

---

## Proposed Changes

### 1. In-Bounds Mobile Clock Rendering (`src/components/GameTable.tsx`)
- Move the clock badge inside the **GameTable top header status bar** (next to round number, table name, and provably fair status) where it is guaranteed to be 100% visible on all mobile screen sizes.
- Also embed a compact live clock badge inside the **Roadmap HUD Header bar** (next to `#Round` and `Road` / `Bets` toggle buttons).
- Remove negative `-top-9` positioning that causes `overflow-hidden` clipping on mobile browsers.

### 2. Styling & Mobile Responsiveness
- Apply responsive font sizing (`text-[9px] xs:text-[10px] sm:text-xs`).
- Use tabular numbers (`font-mono tabular-nums`) with dark glass background (`bg-black/80 backdrop-blur-md border border-amber-500/40`) for sharp contrast on mobile displays.

---

## Verification Plan
1. **Build & Lint Verification**: Run `lint_applet` and `compile_applet`.
2. **Mobile Layout Check**: Ensure clock is clearly visible on mobile screen widths (320px–480px) and desktop layouts.
