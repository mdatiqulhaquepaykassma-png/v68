# Implementation Plan: P2P Lobby Cleanup & Memory Management

## Problem Statement
When switching away from the P2P tab or unmounting the `P2PLobby` component, background intervals, active WebSocket listeners, and transient modal/matchmaking states can persist in memory or cause redundant state updates on unmounted components.

---

## Proposed Changes

### 1. Dedicated WebSocket Listener with Full Teardown (`src/components/P2PLobby.tsx`)
- Add a dedicated `useEffect` hook in `P2PLobby.tsx` that establishes a WebSocket subscription for live P2P events (`ROOM_CREATED`, `ROOM_UPDATED`, `ROOM_CANCELLED`, `ROOM_EXPIRED`, `P2P_MATCHED`, `DUEL_UPDATE`).
- In the return cleanup function:
  - Set `isMounted = false` to prevent late state updates.
  - Explicitly nullify all event handlers (`onopen`, `onmessage`, `onerror`, `onclose`).
  - Close the WebSocket connection cleanly if open or connecting (`socket.close()`).

### 2. Transient State Cleanup on Unmount (`src/components/P2PLobby.tsx`)
- Inside the unmount cleanup function, explicitly reset transient state variables to release heap references:
  - Reset `isMatchmaking` state to `false`.
  - Clear `copiedRoomId`, `successMsg`, `errorMsg`, and `botSpamWarning`.
  - Close all open modal overlays (`showQuickChallengeModal`, `showNotesModal`, `showReportModal`).
  - Clear `selectedOpponent` and `resolvedRoom` objects.

### 3. Verification of Polling Intervals (`src/components/P2PLobby.tsx` & `src/components/OneOnOneArena.tsx`)
- Ensure all `setInterval` calls for room and chat updates cleanly call `clearInterval` on teardown.

---

## Verification Plan
1. **Linting & Compilation**: Execute `lint_applet` and `compile_applet` to confirm zero build or type errors.
2. **Tab Switching Verification**: Verify smooth memory release when toggling between P2P Lobby, Leaderboard, and Game Table views without lingering event listeners or memory leaks.
