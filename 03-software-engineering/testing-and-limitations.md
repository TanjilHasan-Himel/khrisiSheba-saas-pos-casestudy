# 03.4 — Testing Strategy, Verification & Known Limitations
**Document:** QA Verification Report & Boundary Conditions  
**Author:** Tanjil Hasan Himel (Lead Product Architect & Systems Engineer)  
**Classification:** Canonical Public Portfolio Specification  

---

## 1. Test Suite Execution Summary

```
Test Files  14 passed (14)
Tests       304 passed (304)
Duration    ~7.17s across 14 workers
Engine      Vitest 5.1.0 with React Testing Library
```

### Verified Test Matrix

| Test Suite Path | Tests | Focus Area | Status |
| :--- | :---: | :--- | :---: |
| `apps/api/src/modules/seeds/seeds.test.ts` | 5 | SCA seed germination rate limits & lot expiry | `[VERIFIED]` PASS |
| `apps/api/src/modules/fertilizer/fertilizer.test.ts` | 8 | Subsidized quota consumption & gazetted MRP caps | `[VERIFIED]` PASS |
| `apps/api/src/modules/pesticides/pesticides.test.ts` | 6 | WHO hazard band enforcement & antidote mandates | `[VERIFIED]` PASS |
| `apps/api/src/modules/khata/khata.test.ts` | 7 | Double-entry ledger invariance & SMS templates | `[VERIFIED]` PASS |
| `apps/api/src/server.test.ts` | 10 | Fastify routing, health checks, request correlation | `[VERIFIED]` PASS |
| `apps/api/src/modules/operations/operations.test.ts` | 49 | Cash drawer reconciliation & operational expenses | `[VERIFIED]` PASS |
| `apps/api/src/modules/operations/stress-challenge.test.ts`| 73 | High-concurrency race condition & lock stress | `[VERIFIED]` PASS |
| `apps/saas/src/utils/catmullRom.test.ts` | 10 | Spline curve C1 continuity & sub-pixel tangents | `[VERIFIED]` PASS |
| `apps/saas/src/hooks/useGooScale.test.ts` | 11 | Physical volume preservation (scaleY * scaleX^2 = 1.0) | `[VERIFIED]` PASS |
| `apps/saas/src/utils/analyticsMath.test.ts` | 16 | Aging bar segmentations & BDT minor-unit math | `[VERIFIED]` PASS |
| `apps/saas/src/adversarial.test.ts` | 51 | Rapid target reversals (+1000 to -1000) stability | `[VERIFIED]` PASS |
| `apps/saas/src/hooks/useSpring.test.ts` | 9 | Damped harmonic oscillator convergence precision | `[VERIFIED]` PASS |
| `apps/saas/src/components/OperationsModule.test.tsx` | 27 | UI rendering, modal interactions & A4 print flow | `[VERIFIED]` PASS |
| `packages/contracts/src/contracts.test.ts` | 22 | Zod schema validation across all shared DTOs | `[VERIFIED]` PASS |

---

## 2. Known Limitations & Edge Conditions

1. **Direct Telecom SMS Integration:** SMS reminder messages currently format valid Bengali templates and deep links (`wa.me` and device SMS intent). Direct bulk SMS gateway dispatch (via Teletalk / Grameenphone SMS-C) requires merchant SIM/API credentials.
2. **Offline Transaction Finalization:** The current release supports offline draft entry, but authoritative stock decrements and ledger posting require connection to the primary PostgreSQL node to guarantee serializability.
3. **Multi-Currency:** The system exclusively supports Bangladeshi Taka (BDT, ৳) and minor units (Paisa). Multi-currency transactions are deliberately excluded as non-goals.

