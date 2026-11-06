# 04.2 — Engineering Trade-Offs & Evaluated Alternatives
**Document:** Technical Trade-Off Matrix  
**Author:** Tanjil Hasan Himel (Lead Product Architect & Systems Engineer)  
**Classification:** Canonical Public Portfolio Specification  

---

## 1. Trade-Off Analysis Matrix

| Engineering Dimension | Alternative Evaluated | Chosen Approach | Rationale for Choice |
| :--- | :--- | :--- | :--- |
| **Monorepo Strategy** | Polyrepo (Separate git repos for Web, API, and Types) | Turborepo / pnpm workspaces | Monorepo eliminates version drift across shared Zod validation schemas (`@krishi/contracts`) and database types. |
| **API Architecture** | GraphQL (Apollo / Yoga) | Fastify 5 REST API | Fastify delivers 4x lower latency and minimal memory overhead on low-cost single-core cloud nodes compared to heavy GraphQL AST parsing. |
| **Data Fetching State** | React Query / SWR with Polling | Local State + Server-Rendered Props | Avoids aggressive polling load on rural slow 3G cellular connections; guarantees predictable client-side memory usage. |
| **Chart Visualization** | Recharts / Chart.js | Native SVG + Catmull-Rom Spline Engine | Shaved >140KB off the bundle; eliminated React 19 hydration bugs; enabled exact physical spring dampening. |
| **Navigation Shell** | Floating Drawer Navigation | Sticky Viewport Rail (`100dvh` lock) | Eliminates layout shift (CLS); ensures counter cashiers have persistent access to POS, Khata, and Sourcing tabs. |

