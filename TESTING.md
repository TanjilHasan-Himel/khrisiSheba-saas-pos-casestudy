# Krishi Sheba (কৃষি সেবা) — Testing Strategy & Verification Report

## 1. Test Suite Summary

- **Total Test Files**: 14
- **Total Tests**: 304 passed (100% passing)
- **Execution Time**: ~7.17s across 14 workers
- **Test Framework**: Vitest 5.1.0 with React Testing Library

```
 ✓ apps/api/src/modules/seeds/seeds.test.ts (5 tests)
 ✓ apps/api/src/modules/fertilizer/fertilizer.test.ts (8 tests)
 ✓ apps/api/src/modules/pesticides/pesticides.test.ts (6 tests)
 ✓ apps/api/src/modules/khata/khata.test.ts (7 tests)
 ✓ apps/api/src/server.test.ts (10 tests)
 ✓ apps/api/src/modules/operations/operations.test.ts (49 tests)
 ✓ apps/api/src/modules/operations/stress-challenge.test.ts (73 tests)
 ✓ apps/saas/src/utils/catmullRom.test.ts (10 tests)
 ✓ apps/saas/src/hooks/useGooScale.test.ts (11 tests)
 ✓ apps/saas/src/utils/analyticsMath.test.ts (16 tests)
 ✓ apps/saas/src/adversarial.test.ts (51 tests)
 ✓ apps/saas/src/hooks/useSpring.test.ts (9 tests)
 ✓ apps/saas/src/components/OperationsModule.test.tsx (27 tests)
 ✓ packages/contracts/src/contracts.test.ts (22 tests)

 Test Files  14 passed (14)
      Tests  304 passed (304)
```

---

## 2. Test Dimension Breakdown

### A. Mathematical Invariants & Physical Simulations
- **Volume Preservation (`useGooScale`)**: Enforces `scaleY * scaleX^2 = 1.0` during liquid pull deformation.
- **Harmonic Spring Stability (`useSpring`)**: Damped oscillation numerical stability verified against rapid target reversals (+1000 to -1000) without runaway velocity explosion.
- **Catmull-Rom Spline Interpolation (`catmullRom.ts`)**: Monotonicity preservation, tangent vector continuity, and sub-pixel projection precision.

### B. Regulatory & Domain Compliance
- **Gazetted Price Controls**: Rejection of fertilizer sales exceeding gazetted MRP caps.
- **Toxicity Banding Invariants**: Mandatory PPE validation and antidote presence for Class I/II hazard bands before dispensing.
- **SCA Certification Expiry**: Prevention of expired seed lot dispatches.

### C. Financial & Ledger Integrity
- **Double-Entry Balance Equation**: `Paid Amount + Outstanding Due = Grand Total` verified across 10,000 randomized synthetic transactions.
- **Split Payment Allocation**: Zero-sum reconciliation across Cash, MFS (bKash/Nagad), and Seasonal Khata balances.
- **Procurement Reconciliation**: `Received Quantity + Shortage = Ordered Quantity` across partial deliveries.

---

## 3. How to Run Tests

```bash
# Run complete test suite
pnpm test

# Run tests in watch mode
pnpm vitest

# Run type check validation
pnpm typecheck

# Run linting rules
pnpm lint
```
