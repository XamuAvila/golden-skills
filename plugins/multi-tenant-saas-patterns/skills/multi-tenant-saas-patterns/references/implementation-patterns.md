# Implementation Patterns — Multi-Tenant SaaS

Patterns significant during implementation: resource isolation, URL segmentation, and connection pooling.

---

## §4.1 Isolating Filter

**Problem**: A multi-tenant application uses shared resources (caches, config files, log files, static variables). How do you restrict each tenant to only their relevant resources?

**Solution**: Use an **isolating filter** — a mechanism that scopes every shared resource to the current tenant's context. Prefer built-in filter mechanisms over hand-rolled ones.

**Built-in filter examples**:

| Resource | Filter mechanism |
|---|---|
| Database rows | `WHERE tenant_id = ?` in every query |
| Log files | Log4j Appender with tenant-specific file/format |
| Cache entries | Tenant-specific key prefix/namespace |
| File system | Tenant-specific directory or filename prefix |
| XML config | Tenant-specific scope/context within config file |

---

## §4.2 Sub-Domain-Per-Tenant

**Problem**: Segment/isolate the application's interface on a per-tenant basis at the URL level.

**Solution**: Each tenant gets its own sub-domain: `tenant1.example.org`, `tenant2.example.org`.

**SSL considerations**:
- Per-tenant cert: one cert per sub-domain (expensive at scale)
- Wildcard cert: covers all single-level sub-domains of a domain (one cert, higher cost than single-domain)

**Advantages**:
- ✓ First-level tenant identification from URL alone (before auth)
- ✓ Enables tenant-specific IdP selection at the proxy layer
- ✓ Per-tenant UI customization possible at DNS/proxy level
- ✓ Better security: same-origin policy prevents XSS across tenant sub-domains
- ✗ Wildcard SSL cert typically expensive

---

## §4.3 Sub-Directory per Tenant

**Problem**: Need per-tenant URL isolation but cannot use Sub-Domain-Per-Tenant (cost of wildcard SSL cert is prohibitive).

**Solution**: Segment by path: `example.org/tenant1`, `example.org/tenant2`. Variant: tenant ID in query parameter `example.org?c=tenant1`.

**Implications**:
- Application must maintain separate directories/content per tenant
- First-level tenant resolution still possible from the URL path
- Single SSL cert covers all tenants (same domain)

**Consequences**:
- ✓ Single SSL cert for all tenants
- ✓ Tenant-specific UIs and IdPs possible (tenant known from path)
- ✗ Reveals to users that the application is multi-tenant
- ✗ Lower security than sub-domain: same-origin policy doesn't isolate tenants
- ✗ More prone to Predictable Resource Location attacks
- Note: Sub-Domain-Per-Tenant is generally considered more secure

---

## §4.4 Connection-Pool-Per-Tenant

**Problem**: Database connections must be separated per tenant (especially with Database-Per-Tenant §2.3 or Private Schema §2.4).

**Solution**: Maintain a separate JDBC/DB connection pool for each tenant. When a request arrives, resolve the tenant from the Tenant Context and select the corresponding pool.

**Implementation**:
- Define a pool per tenant at startup or lazily on first request
- With private-schema-per-tenant: if the driver supports naming a schema per pool, specify one schema per pool
- The pool map is keyed by tenant ID: `Map<tenantId, ConnectionPool>`

**Consequences**:
- ✓ Flexibility: different databases or DB servers per tenant if needed
- ✓ Resource quotas per tenant: limit connections/queries at the pool level
- ✓ More stringent per-tenant security (tenant can only reach their pool)
- ✓ Per-tenant maintenance: upgrade one tenant's DB without affecting others
- ✗ Memory overhead grows with tenant count (N active pools)

---

## §4.5 Shared Connection Pool

**Problem**: Pool connection resources efficiently for all tenants (Shared Database Tables §2.5 or Private Schema §2.4 use case).

**Solution**: A single shared JDBC/DB connection pool serves all tenants. At runtime, after acquiring a connection, issue `SQL SET SCHEMA <tenant_schema>` (or equivalent) to point the connection at the correct tenant schema before executing queries.

**Example flow**:
1. Request arrives → Tenant Context resolved → tenant schema name known
2. Acquire connection from shared pool
3. `SET SCHEMA tenant_42;`
4. Execute queries (schema-qualified automatically)
5. Return connection to pool (connection may need schema reset)

**Consequences**:
- ✓ Lower administration overhead (one pool to manage)
- ✓ Fewer total connections → lower DB server resource usage
- ✓ Works well with Shared Database Tables (tables are already shared)
- ✗ Application server must use a privileged DB user (access to all schemas)
- ✗ Not all databases support `SQL SET SCHEMA` (check DBMS compatibility)
- ✗ Connection state management: must reset schema before returning to pool

---

## Pattern Interaction Summary

```
Application-Instance-Per-Tenant
  └── (each instance has its own DB, no shared pool needed)

Shared Application
  ├── Tenant Resolver (§3.7) ← Sub-Domain (§4.2) or Sub-Directory (§4.3)
  ├── Tenant Context (§3.5) — bound to thread for request lifetime
  ├── Tenant Context-Based Router (§3.6) — only if routing to different instances
  ├── Isolating Filter (§4.1) — caches, logs, config, files
  ├── Authentication (§3.8) — App Managed / Dir Sync / SAML
  └── DB strategy:
      ├── Database-Per-Tenant → Connection-Pool-Per-Tenant (§4.4)
      ├── Private Schema → Connection-Pool-Per-Tenant OR Shared Pool + SET SCHEMA
      └── Shared Tables → Shared Connection Pool (§4.5) + Basic Table Layout (§3.2)
          └── Custom fields? → Extension Table Layout (§3.3) or Universal Table Layout (§3.4)
```
