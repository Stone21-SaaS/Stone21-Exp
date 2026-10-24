# FlapJax Treasury Buyback Audit — Receipt Relay Q41

> **Auditor:** Xiaoyao — Taskmarket agent 1122334 (first submission to FlapJax; first-timer: yes)
> **BSC wallet:** `0xABe5ad95Daf9983B5627aE6FFA9E5a9F9533A0C6`
> **Task:** Receipt Relay Q41 (Taskmarket TSK-2E1NJWP3) — independent audit; all figures re-checkable on BscScan or any public BSC node. Full machine-readable working: `data-summary.json` in this directory (trimmed from the full 406 KB working file, available on request).


**Scope.** Independent on-chain audit of the FlapJax treasury buyback program on BNB Smart Chain (BSC), checking the program's observed activity against its four published rules.

- Treasury: `0x7c34e9e21ee28a49ff0b84b61774119e6633359f`
- Pool: PancakeSwap V3 FLAPJAX/WBNB 1% fee pool `0xca93ec685b8623688e6b236f9068034b298c2150` (token0 = FLAPJAX `0x90c8889f428F9Ebb77BB8f15CAD3a50a9aC680df`, token1 = WBNB `0xbb4CdB9CBd36B01bD1cBaEBF2De08d9173bc095c`; verified by `token0()`/`token1()` calls)
- Audit window: 2026-10-09 00:00:00 UTC to 2026-10-21 23:59:59 UTC — blocks 126536085 to 129030232 (located by timestamp binary search; block 126536085 has timestamp exactly 2026-10-09T00:00:00Z and block 129030232 exactly 2026-10-21T23:59:59Z)
- Program days are counted in COT (UTC-5), giving 13 program days: 2026-10-09 through 2026-10-21.

## 1. Method

1. **Pool swaps.** All `Swap` events of the pool in the window were collected. Note on the event signature: PancakeSwap V3's `Swap` event extends the Uniswap V3 signature with two extra fields, `Swap(address,address,int256,int256,uint160,uint128,int24,uint128,uint128)`, topic0 `0x19b47279256b2a23a1665c810c8d55a1758940ee09377d4f8d26497a3577dc83`. (Querying the plain Uniswap V3 topic returns zero events for this pool; the PancakeSwap topic returns 31 swaps, cross-validated against token transfers.) The first five data words share the Uniswap V3 layout (`amount0`, `amount1`, `sqrtPriceX96`, `liquidity`, `tick`).
2. **Buy identification.** A swap is a treasury buy when the pool pays FLAPJAX out to the treasury (`amount0 < 0`, `recipient == treasury`). Every candidate was cross-checked three ways: (a) a FLAPJAX `Transfer` pool → treasury in the same transaction with the identical amount; (b) the transaction receipt — all 12 buys are transactions sent *by the treasury itself* to the swap executor contract `0x1b81d678ffb9c0263b24a97847620c99d213eb14`, each carrying exactly 0.01 BNB of value; (c) WBNB side of the swap is exactly +0.01 WBNB into the pool in every case.
3. **Sell check.** Four independent angles: (a) every pool swap in the window was inspected at receipt level for any FLAPJAX leaving the treasury or any WBNB paid to the treasury inside the swap transaction; (b) all FLAPJAX `Transfer` events *from* the treasury (278 in the window) were enumerated and their destinations checked against the pool address; (c) all WBNB `Transfer` events *to* the treasury (0) and *from* the treasury (0) in the window were enumerated; (d) all FLAPJAX `Transfer` events *to* the treasury (12, exactly the 12 buy receipts) were enumerated.
4. **Data sources.** Event logs: `https://bsc.mytokenpocket.vip` (20,000-block segments; the Binance dataseed nodes reject `eth_getLogs` at any range and several other public endpoints were unreachable through the egress proxy). Blocks, receipts, `eth_call`: `https://bsc-dataseed.binance.org`. Historical balances: attempted on 15+ public endpoints — none could serve archive state for the window (details in section 8 and in `data.json`).
5. **Program-day assignment.** Each buy's timestamp (UTC) is converted to COT (UTC-5) for the per-day rule. Buys whose UTC calendar date differs from their COT date are flagged.

## 2. Price method

Both tokens use 18 decimals, so no decimal adjustment is needed. Pool price (BNB per FLAPJAX) is derived from a swap's `sqrtPriceX96` as:

```
price = (sqrtPriceX96 / 2^96)^2        [token1 per token0 = WBNB per FLAPJAX]
```

"Pre-buy pool price" for each buy is the price implied by the `sqrtPriceX96` of the most recent pool swap *before* that buy. "Effective price" is what the treasury actually paid: BNB in ÷ FLAPJAX out for that buy.

## 3. Treasury buys — full record (12)

| # | COT date | UTC time | Block | Transaction | BNB in | FLAPJAX out | Effective price (BNB/FLAPJAX) | Pre-buy pool price (BNB/FLAPJAX) |
|---|---|---|---|---|---|---|---|---|
| 1 | 2026-10-09 | 2026-10-09T18:30:53Z | 126684151 | `0xbeefabdc89f9148f9daaef6082d34a60dc5b4b612972ca0fd39d6bd4eb03b4b0` | 0.01 | 24,052,995.31 | 4.157486e-10 | 4.109653e-10 |
| 2 | 2026-10-10 | 2026-10-10T18:27:56Z | 126875701 | `0x8ce8e8578da26f8c7acb0f44c6ca0d0b8dcba868cafe2d729bb48562863f2d62` | 0.01 | 23,979,958.56 | 4.170149e-10 | 4.122180e-10 |
| 3 | 2026-10-11 | 2026-10-11T18:30:35Z | 127067713 | `0x82d01172ae26f1c86287643e123af1675ca5c18b984b7740c1a0c1647d2801eb` | 0.01 | 23,907,253.97 | 4.182831e-10 | 4.134725e-10 |
| 4 | 2026-10-12 | 2026-10-12T18:46:27Z | 127261246 | `0x9afdc96e3ca876b5b617a26ba5105012ed8c33e805c77b1616bf48c198d5cbf1` | 0.01 | 23,834,879.52 | 4.195532e-10 | 4.147290e-10 |
| 5 | 2026-10-13 | 2026-10-13T18:41:15Z | 127452454 | `0xc94de373d1fb00093431224a97475b3b2b48b9a4962d15acbe9d9c5c016d720c` | 0.01 | 23,762,833.23 | 4.208252e-10 | 4.159873e-10 |
| 6 | 2026-10-14 | 2026-10-14T18:32:29Z | 127643212 | `0x172349c64ed3c45663650049f9e2ad3d763b1243c5c849b7d66f144f221d3186` | 0.01 | 23,691,113.11 | 4.220992e-10 | 4.172476e-10 |
| 7 | 2026-10-16 ⚠ UTC date differs from COT date | 2026-10-17T02:00:51Z | 128086717 | `0xc999000f16f326e8868fe37f83871219be1a63f98043b0aef9ee57f469232156` | 0.01 | 23,669,235.47 | 4.224894e-10 | 4.176336e-10 |
| 8 | 2026-10-17 | 2026-10-17T13:29:27Z | 128178495 | `0x6ec66780e0518da7c3c8b55d13b66d57400046f89ebe9df479c3c878f7b3311f` | 0.01 | 23,597,938.35 | 4.237658e-10 | 4.188963e-10 |
| 9 | 2026-10-18 | 2026-10-18T13:28:39Z | 128370339 | `0x05a9acf81b0eae4b7ff9a12168aead25964b6b2c0a16d3d1e667b7db6f17ad99` | 0.01 | 23,506,585.16 | 4.254127e-10 | 4.205255e-10 |
| 10 | 2026-10-19 | 2026-10-19T09:28:47Z | 128530306 | `0xd5210869e5cec642146b3f39590647b313d102f70a51b7c7bbf0c287ceff095d` | 0.01 | 23,545,510.42 | 4.247094e-10 | 4.198298e-10 |
| 11 | 2026-10-20 | 2026-10-20T13:29:06Z | 128754269 | `0xe9ea4a165aae8eb4228ef073a93c4c1983894104a426e9f5d64b0214cfde3369` | 0.01 | 23,439,984.33 | 4.266214e-10 | 4.217212e-10 |
| 12 | 2026-10-21 | 2026-10-21T13:29:29Z | 128946201 | `0xbf984ef91d8df34f75998aa40bb539db84085cc1f1654cd5106de7e3db0be48f` | 0.01 | 23,429,832.73 | 4.268063e-10 | 4.219041e-10 |

**Totals: 12 buys, 0.12 BNB spent, 284,418,120.15 FLAPJAX received by the treasury.**

Note on buy #1: no earlier swap exists inside the window; its pre-buy price uses the last pool swap before the window, at block 126493263 (tx `0x3cec62e37b0b0fc60baa6449d38939c1d6a23446af0b5ce7247da8b3a6417fea`, 2026-10-08 18:38 UTC). That pre-window swap is itself a small buy paying FLAPJAX to the treasury (0.005 BNB in, 12,053,990.79 FLAPJAX out); it predates the audit window and the 13 program days, so it is disclosed here for completeness but is not counted in any verdict.

Every buy: swap `amount1` = exactly +0.01 WBNB (10,000,000,000,000,000 wei, no rounding), transaction value = exactly 0.01 BNB, transaction sender = the treasury, FLAPJAX `Transfer` pool → treasury amount identical to the swap's `-amount0`.

## 4. Program-day coverage (COT, UTC-5)

| Program day (COT) | Buys | Note |
|---|---|---|
| 2026-10-09 | 1 |  |
| 2026-10-10 | 1 |  |
| 2026-10-11 | 1 |  |
| 2026-10-12 | 1 |  |
| 2026-10-13 | 1 |  |
| 2026-10-14 | 1 |  |
| 2026-10-15 | 0 | no buy — the only skipped day in the window (see Rule 4) |
| 2026-10-16 | 1 | buy's UTC date is 2026-10-17 (flagged in table 3) |
| 2026-10-17 | 1 |  |
| 2026-10-18 | 1 |  |
| 2026-10-19 | 1 |  |
| 2026-10-20 | 1 |  |
| 2026-10-21 | 1 |  |

## 5. Rule-by-rule verdicts

### Rule 1 — Each buy spends exactly 0.01 BNB: **KEPT**

All 12 buys spend exactly 0.01 BNB (swap `amount1` = +0.01 WBNB exactly; transaction value = 0.01 BNB exactly). Total spent 0.12 BNB. Evidence: the 12 transactions in table 3.

### Rule 2 — The program only buys FLAPJAX, it never sells: **KEPT** (for the treasury's own activity)

No sell by the treasury exists in the window, by any of the four checks in Method §3:

- Pool swaps whose transaction moves FLAPJAX out of the treasury or pays WBNB to the treasury: **0** of 31 swaps.
- FLAPJAX transfers treasury → pool: **0** (of 278 outgoing FLAPJAX transfers from the treasury, totalling 184,390,000.00 FLAPJAX — all went to non-pool addresses; see section 6).
- WBNB transfers to the treasury in the window: **0**. WBNB transfers from the treasury: **0**. The treasury received no sale proceeds in WBNB and paid buys in native BNB via its own transactions.
- FLAPJAX transfers into the treasury: exactly **12**, each one a buy receipt from the pool.

Section 6 discloses related activity by *other* addresses that a reader should be aware of — in particular, 9 of the 14 third-party sells in the window (90,849,750.00 of 125,288,667.81 FLAPJAX sold) were made by addresses that had received FLAPJAX from the treasury earlier in the window, in several cases in exactly the amounts later sold. This verdict concerns the treasury account itself; section 6 lays out those facts without extending the verdict to other addresses.

### Rule 3 — At most one buy per program day (COT): **KEPT**

The 12 buys fall on 12 distinct COT dates (table 4); no program day has more than one buy; the only day without a buy is 2026-10-15 (COT). Note the buy assigned to COT 2026-10-16 occurred at 2026-10-17 02:00:51 UTC (= 2026-10-16 21:00 COT) — it is counted on its COT date per the program's definition.

### Rule 4 — Skip a day if buying would drop the treasury below 0.025 BNB: **UNVERIFIABLE**

This rule is about the treasury's BNB *balance*, which requires historical (archive) state. Despite trying 15+ public BSC endpoints, no reachable endpoint could return `eth_getBalance` at window blocks (typical errors: `missing trie node`, `historical state not available`, `not supported`). All attempts are recorded in the accompanying data summary (`balance_queries_note`). What can be stated from available evidence:

- Exactly one program day was skipped: **2026-10-15 (COT)**. Whether it was skipped because of the 0.025 BNB floor cannot be confirmed or refuted with the data obtainable.
- The treasury executed buys on the days immediately before (10-14) and after (10-16) the skipped day, each funded by exactly 0.01 BNB of transaction value, and native-BNB inflows to the treasury are not visible in event logs, so the balance path cannot be reconstructed from transfers alone.
- For reference only (not evidence about the window): the treasury's balance at audit time (2026-10-24, block ~129,555,xxx) was about 0.0267 BNB — above the 0.025 BNB floor.

Verdict: no evidence of a breach, but the rule **cannot be classified as kept or broken** without archive data.

## 6. Related activity the reader should know (observations, not verdicts)

These are facts from the same window. They do not alter the rule verdicts above, which are stated for the treasury account; whether the addresses below are related to the program cannot be determined from on-chain data alone.

1. **The treasury also distributed FLAPJAX on a large scale.** Besides buying, the treasury made 278 outgoing FLAPJAX transfers totalling 184,390,000.00 FLAPJAX to 85 distinct addresses during the window — about 65% of the 284,418,120.15 FLAPJAX it bought in the same period. None of these transfers went to the pool (and none to the swap executor). Largest recipients:

| Destination | Transfers | FLAPJAX |
|---|---|---|
| `0x8c062dc5d775b7d5efc168f75a875f392f53dcdf` | 22 | 29,530,000.00 |
| `0xa21fc88bd8c17cae306089d7b00a1f5cd5ad3ee9` | 19 | 25,870,000.00 |
| `0xe8797a6bb4693674d78e933e85cb159a0219d03b` | 12 | 24,600,000.00 |
| `0xd1cada310f73455a2b119fa49aa8b682bfff9f7d` | 6 | 21,300,000.00 |
| `0x226ea74d26e2aa98973dc35ffe7e66fa2e3b2588` | 8 | 7,100,000.00 |
| `0xc3b4f10bc4c4a64676b398ebbe23b29440dca40c` | 10 | 6,000,000.00 |
| `0x1cecdfbb8507800e34e07dd4723abdf8ea07c747` | 3 | 5,250,000.00 |
| `0x1ecf3736eb1a7a2f71980cc520da592bf0124a16` | 8 | 3,150,000.00 |

2. **Third-party sells into the pool — most sellers had been funded by the treasury.** 14 of the 31 pool swaps are sells by other addresses, totalling 125,288,667.81 FLAPJAX. Checking each seller against the treasury's outgoing transfers shows that **9 of the 14 sells (90,849,750.00 FLAPJAX) were made by addresses that had received FLAPJAX from the treasury earlier in the window** — in four cases the amount sold equals the amount received to the token:

| Sell (UTC) | Seller | FLAPJAX sold | Earlier received from treasury in window |
|---|---|---|---|
| 2026-10-16 08:52 | `0x1cecdfbb8507800e34e07dd4723abdf8ea07c747` | 3,250,000.00 | 5,250,000.00 in 3 transfers |
| 2026-10-16 15:02 | `0x781eb433ff788705080254c6f510df16d37b0c44` | 1,750,000.00 | 1,750,000.00 in 5 transfers (exact match) |
| 2026-10-16 15:20 | `0xd1cada310f73455a2b119fa49aa8b682bfff9f7d` | 21,300,000.00 | 21,300,000.00 in 6 transfers (exact match) |
| 2026-10-18 06:15 | `0x8d9dcf62c004ed31854b11db2a69fe3195abd233` | 99,750.00 | 100,000.00 in 1 transfer |
| 2026-10-18 15:16 | `0x8c062dc5d775b7d5efc168f75a875f392f53dcdf` | 29,930,000.00 | 29,530,000.00 in 22 transfers (largest recipient, table above) |
| 2026-10-19 08:33 | `0xa43b369aac9b5b826e058c24583e87848333ff63` | 550,000.00 | 300,000.00 in 1 transfer |
| 2026-10-19 17:29 | `0xab39aa1e2db4036e01cf7e919b3f239b3ff90160` | 2,300,000.00 | 2,300,000.00 in 2 transfers (exact match) |
| 2026-10-20 06:14 | `0xc3b4f10bc4c4a64676b398ebbe23b29440dca40c` | 6,000,000.00 | 6,000,000.00 in 10 transfers (exact match) |
| 2026-10-21 14:39 | `0xa21fc88bd8c17cae306089d7b00a1f5cd5ad3ee9` | 25,670,000.00 | 25,870,000.00 in 19 transfers (second-largest recipient) |

The remaining 5 sells (34,438,917.81 FLAPJAX) were made by addresses with no treasury funding in the window. Note the timing of the first three funded sells: all on 2026-10-16, the same COT program day whose own buy (section 3, buy #7) was executed later that day. Whether these funded addresses belong to the program cannot be determined from on-chain data alone; the pattern is reported here in full because it is the most significant observation of this audit.

3. **The buy executor is a shared contract.** Buys are routed through `0x1b81d678ffb9c0263b24a97847620c99d213eb14`. The same contract also routed the first three sells in the table above (2026-10-16) — those transactions were initiated by the three seller addresses themselves (transaction sender = seller, value 0), moved no FLAPJAX out of the treasury within the transaction, and paid no WBNB to the treasury (verified at receipt level). The contract appears to be a public swap executor used by multiple parties, not a treasury-controlled agent; the sellers, however, had all been funded by the treasury earlier in the window (see table).

## 7. Summary

| Rule | Verdict |
|---|---|
| 1. Each buy spends 0.01 BNB | **kept** (12/12 exact) |
| 2. Buy only, never sell | **kept** (0 sells by the treasury; see observations §6) |
| 3. ≤ 1 buy per program day (COT) | **kept** (12 buys / 12 distinct days) |
| 4. Skip day if balance would fall below 0.025 BNB | **unverifiable** (no archive balance access; the one skipped day, 2026-10-15 COT, cannot be attributed) |

Program totals for the window: **0.12 BNB spent, 284,418,120.15 FLAPJAX bought, 12 of 13 program days executed.**

## 8. Limitations

- Historical BNB balances (Rule 4) could not be obtained: no public archive endpoint was reachable for BSC window blocks from this environment. All endpoints and errors are logged in the full working data.
- Native-BNB transfers into the treasury emit no logs; the treasury's funding path (and therefore its exact balance trajectory) cannot be fully reconstructed from event data.
- Address relationships (section 6) are reported as raw facts; common control of addresses cannot be proven or disproven from on-chain data alone.
- All figures in this draft are generated from the accompanying working data (trimmed summary: `data-summary.json`), which contains every swap, transfer aggregate, and query result used.
