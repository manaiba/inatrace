# INATrace ↔ AgStack Asset Registry Integration

## Spec, Current State Analysis & Roadmap

*Document Version: February 2026*
*Status: Draft for community review*

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Current Integration State](#2-current-integration-state)
3. [Gap Analysis](#3-gap-analysis)
4. [Asset Registry v2.0 Implications](#4-asset-registry-v20-implications)
5. [Proposed Integration Architecture](#5-proposed-integration-architecture)
6. [Detailed Design Spec](#6-detailed-design-spec)
7. [Implementation Roadmap](#7-implementation-roadmap)
8. [Task Breakdown (TODO)](#8-task-breakdown-todo)
9. [Open Questions](#9-open-questions)

---

## 1. Executive Summary

INATrace has a **partial, one-directional integration** with the AgStack Asset Registry. When a farmer's plot polygon is created or updated in INATrace, the backend calls the Asset Registry's `/register-field-boundary` endpoint to obtain a **GeoID** — a deterministic 64-character alphanumeric hash uniquely identifying that geospatial boundary.

**What works today:**
- ✅ Plot polygon → Asset Registry registration → GeoID stored in INATrace DB
- ✅ GeoID displayed in frontend (web + mobile) with copy-to-clipboard
- ✅ "Open in Whisp" button to view GeoID in [whisp.earthmap.org](https://whisp.earthmap.org) for EUDR deforestation analysis
- ✅ GeoID refresh endpoint (manual re-registration)
- ✅ GeoID export in Excel farmer data exports

**What's missing:**
- ❌ No reverse lookup (GeoID → boundary details from registry)
- ❌ No batch registration (bulk farmer imports don't register plots)
- ❌ No ListID support (v2.0 feature: groups of GeoIDs for cooperatives/value chains)
- ❌ No overlap/deduplication intelligence fed back to users
- ❌ Silent failure mode — registration errors silently return null
- ❌ No GeoID history/versioning when plot boundaries change
- ❌ No automated EUDR risk assessment pipeline
- ❌ No preparation for Asset Registry v2.0 (federated Hub+Node, WKT masking, Points/LineStrings)
- ❌ No shared identity between INATrace and User Registry
- ❌ Integration config is empty by default — effectively disabled for most deployments

---

## 2. Current Integration State

### 2.1 Backend (Java/Spring Boot)

#### Files Involved

| File | Purpose |
|------|---------|
| `AgStackClientService.java` | HTTP client for Asset Registry API |
| `ApiRegisterFieldBoundaryRequest.java` | Request model (`s2_index`, `wkt`) |
| `ApiRegisterFieldBoundaryResponse.java` | Response model (`Geo Id`, `matched geo ids`) |
| `ApiRegisterFieldBoundaryErrorResponse.java` | Error response model |
| `CompanyService.java` | Calls registration on plot create/update |
| `CompanyController.java` | REST endpoint for GeoID refresh |
| `Plot.java` (entity) | Stores `geoId` column in DB |
| `ApiPlot.java` (API model) | Exposes `geoId` in REST responses |
| `application.properties.template` | Config: `INATrace.agstack.{apiKey, clientSecret, baseURL}` |

#### Registration Flow

```
Farmer Plot Created/Updated
    │
    ▼
CompanyService.generatePlotGeoID()
    │
    ├─ Validates ≥ 3 coordinates
    ├─ Ensures polygon is closed (first coord = last coord)
    ├─ Builds WKT: POLYGON ((lng1 lat1, lng2 lat2, ...))
    │
    ▼
AgStackClientService.registerFieldBoundaryResponse()
    │
    ├─ POST {baseURL}/register-field-boundary
    ├─ Headers: API-KEY, CLIENT-SECRET
    ├─ Body: { "s2_index": "8, 13", "wkt": "POLYGON ((...))"}
    │
    ▼
Response
    ├─ Success: { "Geo Id": "abc123...", "matched geo ids": [...] }
    │   ├─ If matched_geo_ids not empty → use first match
    │   └─ Else → use new Geo Id
    │
    └─ Error: silently caught, returns null
```

#### Key Observations

1. **S2 index is hardcoded** to `"8, 13"` — not configurable
2. **Error handling is silent** — `catch (Exception e)` logs error, returns `null`
3. **No retry logic** — if the API is down, the GeoID is permanently null until manual refresh
4. **No authentication token rotation** — uses static API key + client secret
5. **WebClient (reactor)** is used but in blocking mode (`.block()`)

### 2.2 Frontend (Angular)

| Component | GeoID Feature |
|-----------|---------------|
| `MapComponent` | Shows GeoID in plot popup; "Refresh GeoID" button; "Open in Whisp" link |
| `PlotsFormComponent` | Passes GeoID events to parent |
| `PlotsItemComponent` | Displays GeoID with copy-to-clipboard |
| `OpenPlotDetailsExternallyModal` | iFrame to `whisp.earthmap.org` with GeoID |
| Excel export | GeoID column in farmer plot exports |

### 2.3 Mobile (React Native/Expo)

| Component | GeoID Feature |
|-----------|---------------|
| `MapView.tsx` | Displays GeoID in plot details |
| `ListView.tsx` | Shows GeoID; calls `/updateGeoID` endpoint |
| `add-plot.tsx` | Includes GeoID field in plot creation |
| Localization | `geoId` label in en, de, es, rw |

### 2.4 Asset Registry API (v1.0 — Current)

The Asset Registry ([agstack/asset-registry](https://github.com/agstack/asset-registry)) is a Flask app that:

- Accepts WKT polygons via `POST /register-field-boundary`
- Generates deterministic GeoIDs from normalized WKT
- Returns matched GeoIDs if the boundary already exists (above a configurable threshold, default 95%)
- Uses S2 cell indexing for spatial queries
- Requires authentication via user-registry tokens
- Has a 1,000-acre maximum area limit
- Stores boundaries in a PostgreSQL database

---

## 3. Gap Analysis

### 3.1 Critical Gaps

| # | Gap | Impact | Severity |
|---|-----|--------|----------|
| G1 | **Silent failure** — GeoID generation fails silently | Plots appear without GeoIDs; users don't know why | 🔴 High |
| G2 | **No batch registration** — Excel farmer import skips GeoID | Bulk-imported farmers have no GeoIDs until manual refresh | 🔴 High |
| G3 | **Integration disabled by default** — empty config values | Most deployments don't have Asset Registry integration | 🔴 High |
| G4 | **No GeoID validation** — stored GeoIDs are never verified | Stale/invalid GeoIDs persist after boundary changes | 🟡 Medium |
| G5 | **No reverse lookup** — cannot fetch boundary from GeoID | INATrace can't verify what the registry thinks the boundary is | 🟡 Medium |
| G6 | **No overlap intelligence** — matched_geo_ids are not surfaced | Users can't see if their plot overlaps with another registered boundary | 🟡 Medium |
| G7 | **No history/versioning** — old GeoIDs are overwritten | No audit trail when boundaries change | 🟡 Medium |
| G8 | **Hardcoded S2 index** — `"8, 13"` is not configurable | May not be appropriate for all regions/use cases | 🟢 Low |

### 3.2 v2.0 Readiness Gaps

| # | Gap | v2.0 Feature | Severity |
|---|-----|-------------|----------|
| V1 | **No ListID support** | Asset Registry v2.0 introduces ListIDs (hashed sets of GeoIDs) | 🔴 High |
| V2 | **No Hub+Node awareness** | v2.0 uses federated architecture; INATrace hardcodes one URL | 🟡 Medium |
| V3 | **No Point/LineString support** | v2.0 supports Points and LineStrings, not just Polygons | 🟡 Medium |
| V4 | **No WKT masking/permissioning** | v2.0 introduces WKT access controls; INATrace doesn't handle masked responses | 🟡 Medium |
| V5 | **No shared identity** | v2.0 proposes OECD-grade identity via User Registry; INATrace has its own user management | 🟡 Medium |

### 3.3 EUDR Compliance Gaps

| # | Gap | Impact |
|---|-----|--------|
| E1 | **Whisp is view-only** — no automated risk assessment data flows back into INATrace | Compliance officers must manually check each plot in Whisp |
| E2 | **No deforestation risk score stored** — INATrace doesn't persist Whisp/EUDR analysis results | No batch compliance reporting possible |
| E3 | **No automated compliance alerts** — no triggers when a GeoID fails deforestation checks | Risks discovered only during manual review |

---

## 4. Asset Registry v2.0 Implications

The Asset Registry is evolving to a **federated, DNS-inspired global system** (see [asset-registry-2.0-proposal.md](https://github.com/agstack/asset-registry/blob/main/asset-registry-2.0-proposal.md)):

### Key Changes for INATrace

| v2.0 Feature | INATrace Impact |
|--------------|-----------------|
| **Hub + Regional Nodes** | INATrace must discover which Node to call; can't hardcode one URL |
| **WKT Permissioning & Masking** | INATrace may receive masked (S2-cell) boundaries instead of full WKT for unauthorized requests |
| **ListIDs** | INATrace can group all plots for a cooperative/value chain into a single ListID for batch traceability |
| **Points & LineStrings** | INATrace currently only supports Polygons; could extend to field-level point features (e.g., single trees, water points) |
| **OECD Identity** | INATrace user management could delegate to the AgStack User Registry for unified identity |
| **Delegation & Permissions** | Registrations may require Node-specific authorization; API-key model may change |

---

## 5. Proposed Integration Architecture

### 5.1 Target Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                          INATrace Platform                              │
│                                                                         │
│  ┌──────────┐  ┌──────────┐  ┌──────────────┐  ┌────────────────────┐  │
│  │ Frontend  │  │  Mobile  │  │   Backend    │  │  Blockchain        │  │
│  │ (Angular) │  │ (React)  │  │  (Spring)    │  │  (Hyperledger)     │  │
│  └─────┬─────┘  └─────┬────┘  └──────┬───────┘  └─────────┬──────────┘  │
│        │              │               │                    │             │
│        └──────────────┴───────┬───────┘                    │             │
│                               │                            │             │
│                    ┌──────────┴──────────┐                 │             │
│                    │  AgStack Integration │                 │             │
│                    │  Service (NEW)       │                 │             │
│                    │                      │                 │             │
│                    │  • GeoID Manager     │                 │             │
│                    │  • ListID Manager    │◄────────────────┘             │
│                    │  • Registry Client   │  (GeoID on blockchain)       │
│                    │  • Whisp/EUDR Client │                              │
│                    │  • Retry & Queue     │                              │
│                    │  • Health Monitor    │                              │
│                    └──────────┬──────────┘                              │
│                               │                                         │
└───────────────────────────────┼─────────────────────────────────────────┘
                                │
                    ┌───────────┴────────────┐
                    │                        │
          ┌─────────┴──────┐     ┌───────────┴──────────┐
          │  Asset Registry │     │  Whisp / EUDR APIs   │
          │  (Hub + Nodes)  │     │  (earthmap.org)      │
          │                 │     │                       │
          │  • Register     │     │  • Deforestation risk │
          │  • Lookup       │     │  • Compliance score   │
          │  • ListIDs      │     │  • Forest mask data   │
          │  • Validate     │     │                       │
          └─────────────────┘     └───────────────────────┘
```

### 5.2 Key Design Principles

1. **Dedicated integration service** — Extract all Asset Registry logic from `CompanyService` into a standalone service with proper error handling, retry, and queuing
2. **Async by default** — GeoID registration should be async with callback/polling; don't block plot creation
3. **Fail gracefully with visibility** — Show users when GeoID registration is pending/failed, not silent null
4. **ListID-ready** — Support grouping plots into ListIDs from day one
5. **v2.0 compatible** — Design for Hub discovery + Node routing from the start
6. **EUDR pipeline** — Build a data pipeline from Whisp back into INATrace for automated compliance
7. **Configurable** — Feature flags for Asset Registry integration (on/off), with a clear onboarding path

---

## 6. Detailed Design Spec

### 6.1 Phase 1: Fix Current Integration (Backend)

#### 6.1.1 New `AssetRegistryService` (replaces `AgStackClientService`)

```java
@Service
public class AssetRegistryService {

    // Configuration
    private String hubBaseUrl;         // v2.0: Hub discovery URL
    private String apiKey;
    private String clientSecret;
    private boolean enabled;           // Feature flag

    // Core Methods
    ApiGeoIdResult registerBoundary(List<PlotCoordinate> coordinates);
    ApiGeoIdResult lookupGeoId(String geoId);
    ApiGeoIdValidation validateGeoId(String geoId, List<PlotCoordinate> currentCoordinates);
    List<ApiGeoIdResult> registerBoundariesBatch(List<Plot> plots);
    String createListId(List<String> geoIds);

    // Health & Status
    boolean isAvailable();
    AssetRegistryStatus getStatus();
}
```

#### 6.1.2 Registration Status Model

```java
public enum GeoIdStatus {
    PENDING,        // Registration queued
    REGISTERED,     // Successfully registered
    MATCHED,        // Matched existing boundary
    FAILED,         // Registration failed (with error detail)
    STALE           // Boundary changed since registration
}
```

New DB columns on `Plot`:
- `geo_id_status` (enum) — current registration status
- `geo_id_error` (text, nullable) — error message if failed
- `geo_id_registered_at` (timestamp) — when GeoID was obtained
- `geo_id_previous` (varchar, nullable) — previous GeoID for audit trail
- `matched_geo_ids` (text, nullable) — JSON array of matched IDs

#### 6.1.3 Retry & Queue System

```
Plot Created/Updated
    │
    ▼
GeoIdRegistrationQueue (async)
    │
    ├─ Attempt 1 → Success? → Store GeoID, status=REGISTERED
    │
    ├─ Attempt 1 → Failure? → status=PENDING, schedule retry
    │
    ├─ Retry (exponential backoff: 1min, 5min, 30min, 2hr, 24hr)
    │
    └─ Max retries reached → status=FAILED, alert user
```

#### 6.1.4 Batch Registration for Farmer Import

When farmers are imported from Excel:
1. Parse all plot coordinates
2. Queue batch registration job
3. Process in chunks of 50 plots
4. Update GeoIDs as results arrive
5. Generate import report with GeoID status per plot

### 6.2 Phase 2: Frontend & Mobile UX

#### 6.2.1 GeoID Status Indicators

| Status | Frontend Display |
|--------|-----------------|
| `PENDING` | 🟡 Spinner + "Registering..." |
| `REGISTERED` | 🟢 GeoID with copy button + "Open in Whisp" |
| `MATCHED` | 🔵 GeoID + "Matched existing boundary" info icon |
| `FAILED` | 🔴 "Registration failed" + retry button + error detail |
| `STALE` | 🟠 "Boundary changed — re-register?" prompt |

#### 6.2.2 Overlap/Duplicate Alerts

When `matched_geo_ids` is returned:
- Show info banner: "This boundary matches N existing registered boundaries"
- Allow user to view matched GeoIDs
- Option to adopt existing GeoID or register as new

#### 6.2.3 Bulk GeoID Management

New UI section in Company management:
- **"GeoID Status Dashboard"** — overview of all plots and their GeoID status
- Filters: Pending, Registered, Failed, Stale
- Bulk actions: Re-register all failed, Export GeoID report

### 6.3 Phase 3: ListID Integration

#### 6.3.1 Cooperative ListIDs

When Asset Registry v2.0 ListIDs are available:
- Automatically create a ListID for each cooperative (all farmer plots)
- Create ListIDs per value chain
- Store ListID on Company/ValueChain entities
- Use ListID for batch EUDR compliance checks

#### 6.3.2 Data Model

```
Company (cooperative)
  └── listId: String (hash of all plot GeoIDs)
  └── plots[]
        └── geoId: String

ValueChain
  └── listId: String (hash of all plots in chain)
```

### 6.4 Phase 4: EUDR Compliance Pipeline

#### 6.4.1 Automated Whisp Integration

```
GeoID Registered
    │
    ▼
Whisp API Call (async)
    │
    ├─ Deforestation risk score
    ├─ Forest cover percentage
    ├─ Land use classification
    ├─ Compliance status (EUDR)
    │
    ▼
Store in INATrace
    │
    ├─ Plot.eudrRiskScore
    ├─ Plot.eudrComplianceStatus
    ├─ Plot.lastEudrCheck
    │
    ▼
Alerts & Reporting
    ├─ Dashboard with risk heatmap
    ├─ Non-compliant plot alerts
    └─ Batch compliance reports (PDF)
```

### 6.5 Phase 5: Asset Registry v2.0 Migration

#### 6.5.1 Hub Discovery

```
INATrace → Hub: "Which Node handles this coordinate?"
Hub → INATrace: "node-xyz.agstack.org"
INATrace → Node: "Register this boundary"
Node → INATrace: "GeoID: abc123..."
```

#### 6.5.2 WKT Masking Handling

When INATrace receives a masked response (S2 cell instead of full WKT):
- Display approximate area on map
- Show "Restricted — approximate boundary shown"
- Allow user to request full access through Hub permissioning

#### 6.5.3 Identity Federation

When User Registry v2.0 is available:
- Support SSO via AgStack User Registry
- Map INATrace roles to AgStack identity
- Use AgStack credentials for Asset Registry API calls

---

## 7. Implementation Roadmap

### Phase 1: Fix & Harden (0–3 months)
> Priority: 🔴 Critical — fix what's broken

| # | Task | Effort | Priority |
|---|------|--------|----------|
| 1.1 | Replace `AgStackClientService` with `AssetRegistryService` | 2 weeks | 🔴 |
| 1.2 | Add `geo_id_status`, `geo_id_error`, `geo_id_registered_at` to Plot entity | 1 week | 🔴 |
| 1.3 | Implement async registration queue with retry | 2 weeks | 🔴 |
| 1.4 | Add proper error handling (no silent null) | 1 week | 🔴 |
| 1.5 | Enable batch GeoID registration for Excel farmer import | 2 weeks | 🔴 |
| 1.6 | Add configuration validation + health check endpoint | 1 week | 🟡 |
| 1.7 | Make S2 index configurable | 2 days | 🟢 |
| 1.8 | Add integration feature flag (`asset-registry.enabled`) | 2 days | 🟡 |
| 1.9 | Add `geo_id_previous` for audit trail | 3 days | 🟡 |
| 1.10 | Write integration tests with mock Asset Registry | 1 week | 🔴 |

### Phase 2: UX Improvements (3–6 months)
> Priority: 🟡 Important — make it usable

| # | Task | Effort | Priority |
|---|------|--------|----------|
| 2.1 | Frontend: GeoID status indicators (pending/registered/failed/stale) | 1 week | 🔴 |
| 2.2 | Frontend: Overlap/matched GeoID alerts | 1 week | 🟡 |
| 2.3 | Frontend: Bulk GeoID status dashboard | 2 weeks | 🟡 |
| 2.4 | Mobile: GeoID status indicators | 1 week | 🟡 |
| 2.5 | Mobile: Retry button for failed registrations | 3 days | 🟡 |
| 2.6 | GeoID status in Excel export | 2 days | 🟢 |
| 2.7 | Admin notification when batch registration completes | 3 days | 🟢 |

### Phase 3: ListIDs & Grouping (6–9 months)
> Priority: 🟡 Strategic — prepares for v2.0

| # | Task | Effort | Priority |
|---|------|--------|----------|
| 3.1 | Design ListID data model (Company, ValueChain) | 1 week | 🟡 |
| 3.2 | Backend: ListID creation when cooperative plots change | 2 weeks | 🟡 |
| 3.3 | Backend: ListID CRUD API | 1 week | 🟡 |
| 3.4 | Frontend: ListID display on company/value chain pages | 1 week | 🟢 |
| 3.5 | Blockchain: Record ListIDs on Hyperledger for immutable audit | 2 weeks | 🟡 |

### Phase 4: EUDR Compliance Pipeline (9–12 months)
> Priority: 🟡 High value — compliance automation

| # | Task | Effort | Priority |
|---|------|--------|----------|
| 4.1 | Whisp API client service | 2 weeks | 🟡 |
| 4.2 | Automated deforestation risk assessment on GeoID registration | 2 weeks | 🟡 |
| 4.3 | Plot entity: `eudr_risk_score`, `eudr_compliance_status`, `last_eudr_check` | 1 week | 🟡 |
| 4.4 | Frontend: EUDR compliance dashboard with risk heatmap | 3 weeks | 🟡 |
| 4.5 | Compliance alert system (email + in-app) | 1 week | 🟢 |
| 4.6 | Batch EUDR compliance report (PDF generation via ReportingService) | 2 weeks | 🟢 |

### Phase 5: Asset Registry v2.0 Migration (12–18 months)
> Priority: 🟢 Future — when v2.0 is available

| # | Task | Effort | Priority |
|---|------|--------|----------|
| 5.1 | Hub discovery client (resolve coordinates → authoritative Node) | 2 weeks | 🟡 |
| 5.2 | Multi-node support in registration client | 1 week | 🟡 |
| 5.3 | WKT masking handling (detect masked responses, show approximate) | 1 week | 🟡 |
| 5.4 | Point & LineString geometry support | 2 weeks | 🟢 |
| 5.5 | User Registry SSO integration | 3 weeks | 🟢 |
| 5.6 | Permission token flow for WKT access | 1 week | 🟢 |

---

## 8. Task Breakdown (TODO)

### Immediate (Next Sprint)

- [ ] **Audit current integration** — Run INATrace with Asset Registry enabled; confirm API calls work
- [ ] **Document current API contract** — What does the Asset Registry actually return today? Are there undocumented fields?
- [ ] **Set up local Asset Registry** for development — Docker compose with both services
- [ ] **Create `AssetRegistryService`** — New service class with proper structure
- [ ] **Add `geo_id_status` to Plot entity** — DB migration
- [ ] **Add feature flag** — `INATrace.assetRegistry.enabled=false` default

### Short-term (1–3 months)

- [ ] Implement async registration queue (Spring `@Async` or message queue)
- [ ] Replace all `AgStackClientService` calls with new `AssetRegistryService`
- [ ] Add error handling with user-visible status
- [ ] Enable batch registration for farmer Excel import
- [ ] Write integration tests
- [ ] Frontend: GeoID status indicators
- [ ] Mobile: GeoID status indicators
- [ ] Health check endpoint (`/api/integrations/asset-registry/health`)

### Medium-term (3–6 months)

- [ ] Overlap/matched GeoID user alerts
- [ ] GeoID status bulk dashboard
- [ ] GeoID audit trail (previous GeoIDs)
- [ ] Docker compose with INATrace + Asset Registry for demo/development

### Long-term (6–18 months)

- [ ] ListID support (when Asset Registry v2.0 is available)
- [ ] Whisp/EUDR automated compliance pipeline
- [ ] Asset Registry v2.0 Hub+Node migration
- [ ] Shared identity with User Registry
- [ ] Point/LineString geometry types

---

## 9. Open Questions

1. **Is the Asset Registry API stable?** — The v2.0 proposal is in draft. Should INATrace integration work target v1.0 API stability or plan for the v2.0 API from the start?

2. **Authentication model** — Current integration uses static API-KEY + CLIENT-SECRET. Will v2.0 change this to OAuth/JWT via User Registry?

3. **Whisp API availability** — Is there a Whisp API for automated calls, or is it only an interactive web UI? What are rate limits?

4. **GeoID on blockchain** — Should GeoIDs be recorded on the Hyperledger Fabric blockchain for immutable traceability? This would connect the Asset Registry identity to the blockchain audit trail.

5. **Mobile offline registration** — Should the mobile app queue GeoID registrations for when connectivity is available, or always delegate to backend?

6. **Multi-tenant considerations** — In a multi-tenant INATrace deployment, should each tenant have their own Asset Registry credentials, or share a single integration?

7. **Data sovereignty** — With Asset Registry v2.0 regional Nodes, how does INATrace handle the case where different farmers' plots are in different regions with different Nodes?

---

## Appendix A: Current Code References

| Location | File | Lines |
|----------|------|-------|
| Backend | `components/agstack/AgStackClientService.java` | Service class (entire file) |
| Backend | `components/agstack/api/ApiRegisterFieldBoundaryRequest.java` | Request DTO |
| Backend | `components/agstack/api/ApiRegisterFieldBoundaryResponse.java` | Response DTO (`Geo Id`, `matched geo ids`) |
| Backend | `components/company/CompanyService.java` | `generatePlotGeoID()` (L1367–1390), `refreshGeoIDForUserCustomerPlot()` (L1346–1364) |
| Backend | `components/company/CompanyController.java` | `POST /userCustomers/{id}/plots/{plotId}/updateGeoID` (L242) |
| Backend | `db/entities/common/Plot.java` | `geoId` column (L33) |
| Backend | `application.properties.template` | `INATrace.agstack.{apiKey,clientSecret,baseURL}` (L145–147) |
| Frontend | `shared/map/map.component.ts` | GeoID popup, refresh button, Whisp link |
| Frontend | `company/company-farmers/open-plot-details-externally-modal/` | Whisp iframe integration |
| Frontend | `company/company-common/plots-item/plots-item.component.html` | GeoID display + copy |
| Mobile | `components/plots/ListView.tsx` | GeoID display, refresh call (L320) |
| Mobile | `components/plots/MapView.tsx` | GeoID in plot details |
| Mobile | `types/plot.ts` | `geoId` type field |

## Appendix B: Asset Registry API v1.0

**Endpoint:** `POST /register-field-boundary`

**Headers:**
- `API-KEY`: Static API key
- `CLIENT-SECRET`: Static client secret

**Request Body:**
```json
{
  "s2_index": "8, 13",
  "wkt": "POLYGON ((lng1 lat1, lng2 lat2, ...))"
}
```

**Success Response:**
```json
{
  "Geo Id": "64-character-alphanumeric-hash",
  "matched geo ids": ["existing-geo-id-1", "existing-geo-id-2"]
}
```

**Error Response:**
```json
{
  "error": "Error description",
  "message": "Detailed message"
}
```

**Constraints:**
- Maximum area: 1,000 acres
- Polygon must be closed (first coordinate = last coordinate)
- Minimum 3 coordinates (excluding closing coordinate)

