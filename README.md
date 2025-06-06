
### 📝 Comprehensive Audit Checklist for **`pump`** Program

*(Anchor / Solana, pre-sale + Bonding-Curve + Raydium LP)*

> Use this list as a **step-by-step worksheet**.
> Tick every box before main-net deployment to keep audit hours (and therefore cost) as low as possible.

| #                                 | Check Item                                                                                                           | Notes / How to Verify                                  |
| --------------------------------- | -------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------ |
| **A — Project Meta**              |                                                                                                                      |                                                        |
| A-1                               | **Commit hash frozen** for audit scope                                                                               | Tag the exact git commit ✅                             |
| A-2                               | **Upgrade authority** secured (or revoked)                                                                           | If upgradeable, put behind Timelock / Multisig         |
| A-3                               | **IDL generated & published**                                                                                        | `anchor idl init …` so integrators test same interface |
| **B — Global Config / Authority** |                                                                                                                      |                                                        |
| B-1                               | `configure` verifies **admin signer** (`authority == signer`)                                                        |                                                        |
| B-2                               | All PDAs derive from **constant seed prefixes** (`Config::SEED_PREFIX`, `BondingCurve::SEED_PREFIX`)                 |                                                        |
| B-3                               | `toggle_pause` flips `paused` flag and **every user-facing ix** (`launch`, `swap`, etc.) calls `check_not_paused`    |                                                        |
| **C — Launch Instruction**        |                                                                                                                      |                                                        |
| C-1                               | **String length caps** (`name ≤ 32`, `symbol ≤ 10`, `uri ≤ 200`)                                                     |                                                        |
| C-2                               | Replace **`creator_fee: f64`** with **integer basis points** (`u16`)                                                 |                                                        |
| C-3                               | `total_supply` sanity – checked ≥ 1 token & ≤ u64::MAX/100 to avoid overflow                                         |                                                        |
| C-4                               | Donation branch mints **80 % → curve, 20 % → project** (or 100 % → curve) exactly; no zombie ATA when `donate=false` |                                                        |
| C-5                               | Metaplex **metadata PDA** is derived, not arbitrary                                                                  |                                                        |
| C-6                               | After minting, **mint authority revoked** (`set_authority`)                                                          |                                                        |
| C-7                               | Emits `TokenCreated` with bounded URI length                                                                         |                                                        |
| **D — Swap Instruction**          |                                                                                                                      |                                                        |
| D-1                               | `direction` enum validated (0 buy, 1 sell, 2 buyExact) else `InvalidDirection`                                       |                                                        |
| D-2                               | **Result of `.sell()` assigned** to `is_completed`                                                                   |                                                        |
| D-3                               | **Signer-seeds** used in every CPI requiring bonding-curve PDA authority                                             |                                                        |
| D-4                               | `min_out` slippage guard enforced in buy / sell math                                                                 |                                                        |
| D-5                               | Fees (`buy_fee_percent`, `sell_fee_percent`) applied and credited to `fee_recipient` ATA                             |                                                        |
| D-6                               | **Curve completion flag** set once; double-completion impossible                                                     |                                                        |
| D-7                               | Emits `CurveCompleted` event once, with timestamp                                                                    |                                                        |
| **E — Bonding Curve Math**        |                                                                                                                      |                                                        |
| E-1                               | All price & supply calculations use **checked integer math (u128)**                                                  |                                                        |
| E-2                               | No floating-point, no `powf`; use fixed-point where needed                                                           |                                                        |
| E-3                               | Target tokens and curve limit align with minted supply                                                               |                                                        |
| **F — Migration & Liquidity**     |                                                                                                                      |                                                        |
| F-1                               | `migrate` callable **only after `is_completed` == true**                                                             |                                                        |
| F-2                               | Raydium CPI parameters (seed, open\_time) validated; pool not already initialised                                    |                                                        |
| F-3                               | `remove_liquidity` and `burn_lp_tokens` restricted to admin or curve PDA                                             |                                                        |
| F-4                               | LP burn amount checked ≤ LP balance                                                                                  |                                                        |
| **G — Accounts & Rent**           |                                                                                                                      |                                                        |
| G-1                               | Every `init` / `init_if_needed` account marked **rent-exempt**                                                       |                                                        |
| G-2                               | Account space constants (`BondingCurve::LEN`, `Config::LEN`) match struct size                                       |                                                        |
| G-3                               | No **unchecked `AccountInfo` writes** except metadata PDA                                                            |                                                        |
| **H — Events & Logs**             |                                                                                                                      |                                                        |
| H-1                               | All events ≤ 1 KB                                                                                                    | Avoid exceeding BPF log limits                         |
| H-2                               | Consider extra `Swapped` event per trade for analytics                                                               |                                                        |
| **I — Testing & Tooling**         |                                                                                                                      |                                                        |
| I-1                               | **Unit tests** cover every ix branch, including failure paths                                                        |                                                        |
| I-2                               | **Fuzz tests** (`proptest` / Foundry) on price math, overflow                                                        |                                                        |
| I-3                               | **Compute-unit** profile (`solana-test-validator --compute-unit-price`) < 200 k CU                                   |                                                        |
| I-4                               | `cargo audit` & `cargo deny` → 0 critical deps                                                                       |                                                        |
| I-5                               | `anchor verify` passes on devnet                                                                                     |                                                        |
| **J — Security Edge Cases**       |                                                                                                                      |                                                        |
| J-1                               | SOL escrow balances can’t be withdrawn except via intended ix                                                        |                                                        |
| J-2                               | Re-initialisation of PDAs impossible (`realloc` not used unsafely)                                                   |                                                        |
| J-3                               | No privilege escalation via CPI errors (check all `?` returns)                                                       |                                                        |
| J-4                               | All math checked for **division-by-zero** (e.g., pool reserves)                                                      |                                                        |
| **K — Deployment Cost Control**   |                                                                                                                      |                                                        |
| K-1                               | Program size optimised (< 50 KiB) – strip logs, `anchor build -- --release`                                          |                                                        |
| K-2                               | Only **necessary PDAs** initialised in `launch`; avoid unused ATAs                                                   |                                                        |
| K-3                               | Use `init_if_needed` where possible to reuse accounts                                                                |                                                        |
| K-4                               | Pre-calculate rent & upload enough SOL once (no per-user funding)                                                    |                                                        |

---

