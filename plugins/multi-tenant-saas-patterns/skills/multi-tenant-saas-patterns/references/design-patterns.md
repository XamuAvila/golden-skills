# Design Patterns — Multi-Tenant SaaS

Local design concerns: table layouts, tenant context propagation, routing, resolver, and authentication.

---

## §3.1 Private Table Layout

**Problem**: Using Database-Tables-Per-Tenant — how to support per-tenant schema customizations?

**Solution**: Create a copy of each database table for each tenant. Name the table using a tenant-specific value (tenant ID or name): `Account_17`, `Account_35`, `Account_42`.

**Consequences**:
- ✓ No metadata overhead: no discriminator column needed, no tenant-id filtering
- ✓ Full per-tenant schema customization (different columns per tenant)
- ✗ Number of tables = (number of tenants) × (number of app tables) → grows fast

---

## §3.2 Basic Table Layout

**Problem**: How to store data for all tenants in shared tables?

**Solution**: Add a `Tenant_Id` column to every shared table. All tenant data co-exists in the same table; queries filter by `Tenant_Id`.

```sql
-- Example: shared Employee table
SELECT * FROM Employee WHERE Tenant_Id = 42;
```

**Discussion**: The discriminator column must be enforced by the application (or ORM) for every read and write. Hibernate, JPA EclipseLink, and JPA 2.1 natively support entity-level discriminators.

---

## §3.3 Extension Table Layout

**Problem**: Support table extensions for tenants where multiple tenants share a common extension type, and there are several extension types.

**Solution**: Hybrid of Basic Table Layout + Private Table Layout:
- **Base table** (`Account_Ext`): columns common to all tenants + tenant/row reference
- **Extension tables** (`HealthcareAccount`, `AutomotiveAccount`): hold extension-specific columns, shared by all tenants that need that extension type

```
Account_Ext (Tenant, Row, Aid, Name)
  ← joined with →
HealthcareAccount (Tenant, Row, Hospital, Beds)   -- Tenant 17 only
AutomotiveAccount (Tenant, Row, Dealers)           -- Tenant 42 only
```

**Consequences**:
- ✓ Better table consolidation than Private Table Layout
- ✓ Extension tables only read when referenced in query (no overhead for irrelevant tenants)
- ✗ Many JOINs needed to reconstruct logical source table → may be slow for Edit/View pages
- ✗ Growth of extension types still leads to table proliferation

---

## §3.4 Universal Table Layout

**Problem**: Need maximum schema flexibility in a shared table structure.

**Solution**: A single "universal" table with generic typed columns (`Col1`…`ColN`, typically `VARCHAR`) plus `Tenant` and `Table` identifier columns. Every tenant's data for every logical table maps into this structure.

```
Universal(Tenant, Table, Col1, Col2, Col3, Col4, Col5, Col6)
  Tenant 17, Table 0 → [Acme, St.Mary, 135, -, -, -]
  Tenant 35, Table 1 → [Ball, -, -, -, -, -]
```

**Consequences**:
- ✓ Maximum extensibility: any tenant can use any subset of columns
- ✓ Good consolidation: all data in one table
- ✗ Large number of `NULL` values → sparse table
- ✗ Generic string column type → DBMS indexing features mostly unusable
- ✗ Loss of type safety; boolean, integer must be serialized to string

---

## §3.5 Tenant Context

**Problem**: Components throughout a multi-tenant application need access to tenant/user information (ID, config, auth data) without polluting every method signature.

**Solution**: A **context object** encapsulates all necessary tenant information and is shared through the application for the lifetime of a request (or session). A **Tenant Context Manager** creates, retrieves, and destroys context objects.

**What the context may hold**:
- Tenant ID (minimum)
- Authentication info / user identity
- Tenant status (active, suspended, trial)
- Reference to tenant configuration service
- Feature flags per subscription tier

**Consequences**:
- ✓ Decouples domain logic from non-domain tenant bookkeeping
- ✓ Single source of truth for tenant identity within a request
- Note: typically bound to the current thread (ThreadLocal in Java)

---

## §3.6 Tenant Context-Based Router

**Problem**: Route incoming requests to the correct channel/service/application instance for that tenant.

**Solution**: Derived from the Enterprise Integration Patterns content-based router. Read the **Tenant Context** and route the request to the configured endpoint for that tenant.

**Discussion**:
- Configuration mapping: `tenant_id → route/endpoint`
- Supports different workflows per tenant (each tenant served differently for the same API call)
- Load balancing can be layered on top: route high-load tenants to specific instances
- Routes must be exhaustive and machine-readable
- When used with Application-Instance-Per-Tenant (§2.1), routes to the correct VM/instance

**Consequences**:
- ✓ Complete isolation in request processing per tenant
- ✓ Enables per-tenant SLAs and workflow customization

---

## §3.7 Tenant Resolver

**Problem**: Before any routing or context creation, the system must identify which tenant sent the request.

**Solution**: Extract tenant identity from the request at the application's entry point. Sources:

| Source | Mechanism |
|---|---|
| URL sub-domain | `tenant1.example.org` → extract `tenant1` |
| URL path | `example.org/tenant1` → extract from path segment |
| Query parameter | `example.org?c=tenant1` |
| HTTP host header | Custom header with tenant ID |
| Authentication | Post-auth token contains tenant claim (downside: UI is generic until auth) |

**Authentication-based resolution caveats**:
- UI is generic until the user authenticates (no tenant branding at login page)
- Hard to support multiple IdPs: you need to know the tenant BEFORE selecting which IdP to use
- Workaround: ask for tenant code during login in addition to username

---

## §3.8 Authentication Patterns

### Application Managed Authentication

The SaaS application maintains its own user directory and manages credentials.

- ✓ Simple to implement
- ✗ Provider owns credential responsibility (outside their domain)
- ✗ Users need another set of credentials
- ✗ On-boarding/off-boarding users, migration, and de-provisioning are harder

### Directory Synchronization

The application's internal user directory is synced from each tenant's user directory. Internal directory used for auth.

- ✓ Tenant users authenticate with existing corporate credentials
- ✗ Sync process becomes complex and error-prone as tenant count grows
- ✗ Administrators must manage the sync pipeline

### Customer-Delegated Authentication (SAML / SSO / Federation)

Tenant's local Identity Provider (IdP) issues tokens (e.g., SAML assertions). The SaaS provider validates these tokens instead of managing passwords.

- ✓ Provider never touches user passwords
- ✓ True SSO: users log in once with corporate credentials
- ✓ Tenant controls user lifecycle (provision, deprovision)
- ✗ Higher setup complexity: trust relationship between tenant IdP and SaaS provider
- ✗ Still requires basic account management (mapping tenant user → SaaS account)
- Note: also called "directory federation" or "token-based SSO"
