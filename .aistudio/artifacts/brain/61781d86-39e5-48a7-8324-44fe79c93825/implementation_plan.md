# Implementation Plan: Remove Card Shadows & Glows

## Analysis
The user requested to "remove card's white shadow" (`card er white shadow remove kore dan`). This refers to the prominent outer shadows (`shadow-2xl` / `shadow-xl`) and the white iridescent holographic shine sweeps rendered on the card elements.

---

## Proposed Changes

### 1. Remove Shadows in `PlayingCard.tsx`
- Remove `shadow-2xl` and `shadow-xl` from the card front and card back container elements.
- Simplify card front class list to use zero shadows.
- Remove the holographic iridescent white shine sweep overlay (`bg-gradient-to-r with white/70`) which creates white reflection/shadow overlays.

---

## Verification Plan
1. **Build & Lint Check**: Run `lint_applet` and `compile_applet`.
2. **Visual Check**: Cards will render clean and flat with no white outer shadows or sweeps.
