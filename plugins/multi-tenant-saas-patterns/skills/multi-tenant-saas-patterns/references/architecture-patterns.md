# Architecture Patterns — Multi-Tenant SaaS

Coarse-grained patterns that address non-local design concerns (apply to most/all of the application).

---

## §2.1 Application-Instance-Per-Tenant

**Problem**: Host on shared infrastructure without sharing the application instance (legacy cost, isolation requirements, or fast time-to-market).

**Solution**: Deploy a separate application instance (+ DB) per tenant on shared hardware/OS/hypervisor.

**Variants**:
- *VM-per-tenant*: instance runs inside a VM for added isolation and vertical scalability
- *Packaged Application Instance (Virtual Appliance)*: pre-packaged OS + app image, simplifies provisioning

**Key components** (in a reference mediation layer):

| Component | Responsibility |
|---|---|
| Tenant Identifier | Extracts tenant identity from URL or payload |
| Tenant Authenticator | Authenticates the client |
| Tenant Context Handler | Looks up tenant metadata, binds to current thread as Tenant Context |
| Tenant Resolver | Determines which application instance to route to |
| Tenant Context-Based Router | Routes the request to the resolved instance |

**Consequences**:
- ✓ Easy to SaaS-ify existing applications (minimal code change)
- ✓ Strong isolation: data leakage, performance, security are contained per tenant
- ✓ Per-tenant maintenance: upgrades, backups, migrations can be staggered
- ✓ Per-tenant customization (field names, UI, business logic) without shared-app complexity
- ✗ Scalability limit: N tenants = N application + DB instances
- ✗ High cost (no resource pooling); hard to leverage economies of scale
- ✗ Upgrade cycles are long and error-prone (each tenant upgraded separately)

---

## §2.2 Shared Application

**Problem**: Leverage economies of scale — a single application instance serves all tenants.

**Solution**: Use one shared application parameterized with a Tenant Context that customizes behavior per tenant (APIs, GUIs, features, DB routing, config, logs).

**Discussion**:
- The application must be **parameterized** everywhere: interfaces (API + GUI), enabled features (subscription tier), DB connection, config files, log files.
- The database is NOT automatically shared — DB strategy (§2.3–2.5) is an independent decision.
- Customizations per tenant require one of: metadata-driven architecture (§2.6), dependency injection, AOP, or conditional logic keyed on Tenant Context.

**Consequences**:
- ✓ Lower cost; resource pooling across tenants
- ✓ Single-version upgrades: all tenants migrate simultaneously
- ✓ Single application codebase to maintain
- ✗ Converting existing on-premise app requires significant rework
- ✗ Weak tenant isolation by default: queries from one tenant may impact others
- ✗ Per-tenant maintenance (backups, migration) is harder
- ✗ All tenants must upgrade at once (may not be feasible)
- ✗ Per-tenant customizations are harder to design

---

## §2.3 Database-Per-Tenant

**Problem**: Need strong data isolation per tenant (regulatory, banking, medical) or per-tenant backup/restore.

**Solution**: Each tenant's data lives in its own database (regardless of whether the app is shared or per-instance).

**Options**:
- *Dedicated DB Server per tenant* — maximum isolation, highest cost
- *Shared DB Server, separate databases* — balanced isolation

**Discussion**:
- Requires Connection-Pool-Per-Tenant (§4.4) when used with Shared Application.
- Shared data (cross-tenant) lives in a separate shared database with its own pool.
- Also known as "Shared Machine" or "Separate Database" in other literature.

**Consequences**:
- ✓ Easy to extend data model per tenant
- ✓ Simple per-tenant backup/restore (tenant-specific DB objects)
- ✓ Can partition databases onto different disks; avoids table growth limits
- ✓ Different data model versions per tenant; isolated DB upgrades
- ✗ Large tenant count → large number of databases to manage
- ✗ No DB-level resource pooling → higher hardware costs
- ✗ Schema changes must be applied to every tenant's DB
- ✗ Per-tenant upgrades must be processed individually

---

## §2.4 Database-Tables-Per-Tenant / Private Schema

**Problem**: Isolate tenant data without spinning up a full DB per tenant (consolidate DB infrastructure).

**Solution**: Tenants share a DB server but each tenant has **private tables** (Private-Tables-Per-Tenant on a shared schema) or a **private schema** (Private-Schema-Per-Tenant).

**Discussion**:
- Shared schema vs. private schema: for most DBMSs there is little difference in resource overhead (schemas are lightweight prefixing).
- Access control challenge: on a shared DB, a DB user may have access to all schemas. Explicit access grants or application-layer filtering required.
- Physical tablespaces per tenant can improve I/O load balancing and data migration.
- A "table template" can simplify provisioning of new tenants.
- Connection pooling options:
  - *Connection-Pool-Per-Tenant* (§4.4): select pool based on Tenant Context ID
  - *Cross-tenant shared pool* (§4.5): use `SQL SET SCHEMA` at runtime

**Consequences**:
- ✓ Middle ground between Database-Per-Tenant isolation and Shared Database Tables consolidation
- ✓ Easy to extend data model per tenant when needed
- ✗ High tenant count → large number of tables; maintenance overhead grows
- ✗ Resource contention more likely than Database-Per-Tenant
- ✗ Harder to restore individual tenant from backup (need temporary server + import)

---

## §2.5 Shared Database Tables

**Problem**: Maximize resource pooling; all tenants share the same tables (and DB server).

**Solution**: Single set of database tables for all tenants. A **tenant discriminator column** (`tenant_id`) in every table identifies which tenant owns each row. Filter all queries with `WHERE tenant_id = ?`.

**Discussion**:
- Filter reads AND writes (inserts, updates, deletes) to prevent accidental cross-tenant mutation.
- ORM support: Hibernate `@TenantDiscriminator(column="tenant_id")`, JPA 2.1, EclipseLink.
- Table partitioning by `tenant_id` can improve query optimizer efficiency.
- Shared DB → connection pool can be shared too (§4.5 Shared Connection Pool).
- Per-tenant custom fields require a table layout strategy (§3.1–3.4).

**Consequences**:
- ✓ Best resource pooling of all DB strategies
- ✓ Scalability limited only by DB row/column limits
- ✓ Cross-tenant analytics/reporting is easy
- ✓ All tenants backed up together (simple backup ops)
- ✗ Difficult to provide per-tenant offsite backup copy
- ✗ Per-tenant restore is complex (delete rows for tenant, re-import from temp DB)
- ✗ Weakest isolation: one tenant's queries/locks can affect others
- ✗ Schema changes affect all tenants simultaneously
- ✗ Tenant migration requires live queries against operational system

---

## §2.6 Metadata-Driven Architectures

**Problem**: Support per-tenant extensions to objects (UI, APIs, workflows, business logic, data fields) in a shared application without code changes.

**Solution**: Partition application objects into base (invariant) and variable (customizable) parts. Customizations exist only as **metadata** in a Universal Data Dictionary (UDD). At runtime, virtual application components are generated by materializing UDD metadata (with caching for performance).

**Customizable objects**:
- GUIs, APIs, forms: look-and-feel, field names, API versions
- Configuration: policies, encryption keys, PKI certs, namespaces
- Workflows
- Business logic and rules: e.g., discount conditions
- Data schemas and fields: tenant-specific tables, columns, relationships

**Consequences**:
- ✓ Tenant-specific customizations via configuration, not code changes — critical at scale
- ✗ High application complexity: generic runtime execution engine, metadata management
- ✗ Overkill for minor customizations; alternatives: dependency injection, AOP, generative programming, Isolating Filter (§4.1), GUI templates
