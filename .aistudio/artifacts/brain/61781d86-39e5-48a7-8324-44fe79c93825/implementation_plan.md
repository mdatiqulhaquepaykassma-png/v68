# Revised Implementation Plan: Fix Card Deal Animation Jumping & Flickering

## Overview
Standardize stable key props for `PlayingCard` inside `GameTable` and implement a pure, hardware-accelerated 3D card flip animation in `PlayingCard.tsx` without conflicting CSS opacity transitions.

## Proposed Changes

### 1. Pure 3D Hardware Flip (`src/components/PlayingCard.tsx`)
- Remove conflicting CSS transitions on card faces.
- Use 3D `rotateY: flipped ? 180 : 0` and native `backfaceVisibility: "hidden"`.
- Smooth the spring dynamics.

### 2. Stable Component Keys (`src/components/GameTable.tsx`)
- Replace dynamic key suffixes with static, robust keys (`dragon-card`, `tiger-card`) to prevent unwanted React unmounts mid-animation.
