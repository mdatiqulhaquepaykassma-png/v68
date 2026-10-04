# Implementation Plan: Memory Leak Patch & Memory Optimization for Web Service

## Problem Statement
The Web Service instance exceeded its memory limit and was automatically restarted by the hosting container. Analysis of the backend `server.ts` revealed several root causes:

### Root Causes
1. **Unbounded In-Memory Arrays**:
   - `user.transactions`: Appended on every bet, refund, win, loss, deposit, withdrawal across 15+ endpoints without length caps.
   - `userBetHistories`: Continuous growth per user across rounds.
   - `globalTransactions`, `cashierDeposits`, `cashierWithdrawals`, `adminGrants`, `playerReports`, `roundDisputes`, and `chatMessages` growing indefinitely.
2. **Infrequent Garbage Sweep**:
   - The memory sweep timer was running only every 5 minutes (`300000ms`), allowing memory spikes during peak traffic.
3. **High-Frequency Polling Allocation Pressure**:
   - High-frequency polling on `/api/leaderboard`, `/api/metrics`, and `/api/rooms` re-allocated and serialized dynamic JSON strings on every request without micro-caching.
4. **WebSocket Zombie Socket Retention**:
   - Incomplete cleanup on disconnected/stale client sockets.

---

## Proposed Changes & Patches

### 1. Unified Safe Transaction Appender & Array Capping (`server.ts`)
- Introduce `addTransactionToUser(user, tx)` helper that strictly caps `user.transactions` to 40 items.
- Ensure all transaction additions route through this helper.
- Enforce strict bounds on `userBetHistories` (max 40), `globalTransactions` (max 100), and `chatMessages` (max 60).

### 2. Aggressive 60-Second Memory Sweep & GC Watchdog (`server.ts`)
- Shorten sweep interval from 5 minutes to 60 seconds (`60000ms`).
- In each sweep:
  - Prune all per-user transaction and bet history arrays.
  - Purge expired duels, anti-spam rate trackers, and old logs.
  - Terminate dead WebSocket sockets.
  - Monitor `process.memoryUsage()`. If heap usage exceeds threshold (350 MB), perform emergency array trimming and trigger `global.gc?.()`.

### 3. Lightweight Response Micro-Cache (1s TTL) (`server.ts`)
- Add in-memory 1-second cached responses for high-frequency read endpoints (`/api/leaderboard`, `/api/metrics`, `/api/rooms`).
- Drastically reduces GC garbage allocation during multi-user traffic spikes.

---

## Verification Plan
1. **Lint & Compilation**: Run `lint_applet` and `compile_applet` to confirm 0 TypeScript/build errors.
2. **Server Memory Verification**: Check startup and simulated tick cycles to confirm bounded memory footprint.
