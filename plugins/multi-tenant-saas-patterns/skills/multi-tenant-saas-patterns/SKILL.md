---
name: multi-tenant-saas-patterns
description: >
  Use when designing or evaluating multi-tenant SaaS applications. Triggers on: choosing between
  dedicated vs. shared application instances, database isolation strategies (database-per-tenant,
  private schema, shared tables), tenant discriminator columns, table layout for shared schemas
  (basic/extension/universal/private table layout), tenant context propagation, tenant resolution
  from URL or auth, request routing per tenant, authentication (SSO, SAML, directory federation,
  app-managed auth), connection pool isolation vs. shared pools, isolating filters, sub-domain vs.
  sub-directory per tenant, metadata-driven customizations, or trade-offs between tenant isolation,
  resource pooling, and scalability in SaaS architectures.
---

# Multi-Tenant SaaS Patterns

**Source**: Sharda, Chaudhary, Pillai, Gururao — "Patterns of Multi-Tenant SaaS Applications" (EMC iCOE, 2014)

Multi-tenancy = a single shared system instance serves multiple customers (tenants). Enables SaaS economics (resource pooling, single-version upgrades) but introduces isolation complexity.

## Quick Navigation

| Question | Pattern |
|---|---|
| Dedicated vs. shared app per tenant? | `references/architecture-patterns.md` → §2.1–2.2 |
| One DB or shared DB per tenant? | `references/architecture-patterns.md` → §2.3–2.5 |
| Per-tenant customization across shared app? | `references/architecture-patterns.md` → §2.6 |
| Table layout for shared schemas | `references/design-patterns.md` → §3.1–3.4 |
| Propagating tenant info through the app | `references/design-patterns.md` → §3.5 |
| Routing requests to the right endpoint/instance | `references/design-patterns.md` → §3.6 |
| Identifying which tenant sent a request | `references/design-patterns.md` → §3.7 |
| Authentication (SSO, SAML, directory sync) | `references/design-patterns.md` → §3.8 |
| Restricting per-tenant resource access | `references/implementation-patterns.md` → §4.1 |
| Sub-domain vs. sub-directory per tenant | `references/implementation-patterns.md` → §4.2–4.3 |
| DB connection pool per-tenant or shared | `references/implementation-patterns.md` → §4.4–4.5 |

## Master Decision Framework

### Application Topology

```
Need to host on shared infrastructure?
├── Can't share the application code/instance? (legacy, time-to-market, strict isolation)
│   └── Application-Instance-Per-Tenant (VM-per-tenant)
│       ✓ Strong isolation, easy per-tenant maintenance
│       ✗ High cost, poor scalability, long upgrade cycles
└── Can share the application?
    └── Shared Application
        ✓ Lower cost, single-version upgrades
        ✗ Must parameterize everything with Tenant Context
        └── Choose DB strategy independently (§2.3–2.5)
```

### Database Topology

| Strategy | Isolation | Resource Pooling | Maintenance Overhead | Best When |
|---|---|---|---|---|
| Database-Per-Tenant | Strongest | Lowest | High (N databases) | Regulatory, banking, medical |
| Database-Tables-Per-Tenant (Private Schema) | Medium | Medium | Medium | Moderate isolation needs |
| Shared Database Tables | Weakest | Highest | Lowest | High tenant count, cost-sensitive |

### Table Layout (within Shared Database Tables)

| Layout | Flexibility | Performance | Overhead | Use When |
|---|---|---|---|---|
| Basic Table Layout | None | Best | None | All tenants have identical schema |
| Private Table Layout | Full | Good | Many tables | Few tenants, distinct schemas |
| Extension Table Layout | Partial | Medium (JOINs) | Extension tables | Subsets of tenants share extensions |
| Universal Table Layout | Maximum | Poor (sparse/no index) | Null values | Maximum flexibility needed |

### Authentication

| Pattern | Credential ownership | Complexity | Best when |
|---|---|---|---|
| App Managed Authentication | SaaS provider | Low | Simple, few tenants, no enterprise IdP |
| Directory Synchronization | Tenant's own IdP (synced) | Medium | Tenant wants SSO but no SAML support |
| Customer-Delegated (SAML/SSO) | Tenant's local IdP | High | Enterprise tenants, multiple IdPs |

### URL / Tenant Segmentation

| Pattern | Security | Cost | Reveals multi-tenancy |
|---|---|---|---|
| Sub-Domain-Per-Tenant | Higher (same-origin policy) | Wildcard SSL cert | No |
| Sub-Directory per Tenant | Lower (XSS risk) | Single SSL cert | Yes |
