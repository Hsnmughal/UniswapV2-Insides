# 🔄 UniswapV2-Insides

This repository explores the mathematical foundations behind Uniswap V2's core mechanisms. Each section provides both the final formulas and their complete derivations.

## 📊 Table of Contents
- [Swap Math](#swap-math)
- [Swap Fee Calculation](#swap-fee-calculation)
- [Liquidity Operations](#liquidity-operations)
  - [Adding Liquidity](#adding-liquidity)
  - [Pool Shares Minting](#pool-shares-minting)
  - [Removing Liquidity](#removing-liquidity)
  - [Pool Shares Burning](#pool-shares-burning)
- [Flash Swap Mechanics](#flash-swap-mechanics)
- [TWAP Calculation](#twap-calculation)

---

## Swap Math

### Basic Swap Formula
Calculates output token amount (dy) when swapping input tokens (dx) in a constant product AMM.

**Final Equation:** `dy = (Y₀ * dx) / (X₀ + dx)`

**Derivation:**
1. Initial pool state: X₀ and Y₀ tokens
2. After swap: (X₀ + dx) and (Y₀ - dy)
3. Constant product invariant: X₀ * Y₀ = (X₀ + dx) * (Y₀ - dy)
4. Solving for dy: X₀ * Y₀ = (X₀ + dx) * (Y₀ - dy)
5. Y₀ - dy = (X₀ * Y₀) / (X₀ + dx)
6. dy = Y₀ - (X₀ * Y₀) / (X₀ + dx) = (Y₀ * dx) / (X₀ + dx)

---

## Swap Fee Calculation

Accounts for the trading fee when calculating output amounts.

**Final Equation with Fees:** `dy = dx(1-F) * Y₀ / (X₀ + dx(1-F))`

Where:
- dx = Input amount of token X
- dy = Output amount of token Y
- X₀ = Amount of token X in pool
- Y₀ = Amount of token Y in pool
- F = Fee rate (0.003 or 0.3% in Uniswap V2)

**Key Insight:**
- Fee is calculated as: `swap fee = F * dx`
- Only `dx(1-F)` of the input amount participates in the swap

**Example:**
In a pool with 6,000,000 DAI and 3,000 ETH, swapping 1,000 DAI yields approximately 0.498341 ETH after the 0.3% fee.

---

## Liquidity Operations

### Adding Liquidity

To add liquidity without price impact, tokens must be added in the same proportion as they exist in the pool.

**Final Equation:** `dy/dx = Y₀/X₀`

**Derivation:**
1. Current price ratio: `Price = Y₀/X₀`
2. To maintain the same price: `(Y₀ + dy)/(X₀ + dx) = Y₀/X₀`
3. Cross-multiplying: `X₀(Y₀ + dy) = Y₀(X₀ + dx)`
4. Expanding: `X₀Y₀ + X₀dy = Y₀X₀ + Y₀dx`
5. Simplifying: `X₀dy = Y₀dx`
6. Therefore: `dy/dx = Y₀/X₀`

### Pool Shares Minting

Calculates LP tokens to mint when adding liquidity.

**Final Equation:** `S = (L₁ - L₀)/L₀ * T = (dx/X₀) * T = (dy/Y₀) * T`

Where:
- S = Shares to mint
- T = Total existing shares
- L₁ = Liquidity after adding tokens
- L₀ = Liquidity before adding tokens

**Derivation:**
1. The ratio of new to old shares equals the ratio of new to old pool value: `(T + S)/T = L₁/L₀`
2. Solving for S: `S = T * (L₁ - L₀)/L₀`
3. The relative increase in liquidity equals the relative amount of tokens added: `(L₁ - L₀)/L₀ = dx/X₀ = dy/Y₀`

### Removing Liquidity

Calculates token amounts to withdraw when burning LP tokens.

**Final Equations:**
- `DX = X₀ * S/T`
- `DY = Y₀ * S/T`

Where:
- DX, DY = Amounts of tokens X and Y to withdraw
- S = Shares being burned
- T = Total shares outstanding
- X₀, Y₀ = Token amounts in the pool

**Derivation:**
1. The decrease in liquidity is proportional to shares burned: `(L₀ - L₁)/L₀ = S/T`
2. This equals the proportional decrease in each token: `S/T = DX/X₀ = DY/Y₀`

**Key Principle:**
When removing liquidity, the price ratio must remain unchanged: `dy/dx = Y₀/X₀`

This can be derived by setting the price before equal to the price after: `(Y₀ - dy)/(X₀ - dx) = Y₀/X₀`

### Pool Shares Burning

Calculates the overall value withdrawn when burning LP tokens.

**Final Equation:** `L₀ - L₁ = (S/T) * L₀`

**Derivation:**
1. The ratio of remaining to original shares equals the ratio of remaining to original liquidity: `(T - S)/T = L₁/L₀`
2. Solving for withdrawn value: `L₀ - L₁ = L₀ - L₀ * (T - S)/T = L₀ * [1 - (T - S)/T] = L₀ * [S/T]`

---

## Flash Swap Mechanics

Calculates the minimum fee required for flash swaps.

**Final Equation:** `fee ≥ (0.003/0.997) * dx₀`

Where:
- dx₀ = Amount borrowed
- fee = Additional amount to repay

**Derivation:**
1. Pool constraint: `x₀ - dx₀ + 0.997 dx₁ ≥ x₀`
2. Repayment relationship: `dx₁ = dx₀ + fee`
3. Simplifying: `-dx₀ + 0.997 dx₁ ≥ 0`
4. Substituting: `0.997(dx₀ + fee) ≥ dx₀`
5. Solving for fee: `0.997 fee ≥ 0.003 dx₀`
6. Therefore: `fee ≥ (0.003/0.997) * dx₀`

---

## TWAP Calculation

Calculates Time-Weighted Average Price (TWAP) efficiently using cumulative price.

**Final Equation:** `TWAP from t_k to t_n = (C_n - C_k) / (t_n - t_k)`

Where:
- C_j = Cumulative price at time j
- t_k, t_n = Start and end times for the TWAP calculation

**Derivation:**
1. Define cumulative price: `C_j = Σ(i=0 to j-1) Δt_i × p_i`
   - This is the sum of price × duration products up to time j
2. The sum of price changes from t_k to t_n equals `(C_n - C_k)`
3. Dividing by the time period gives the time-weighted average

**Implementation Advantage:**
This approach only requires storing a single cumulative price state variable that updates with each price change, making TWAP calculations gas-efficient and practical for on-chain implementation.