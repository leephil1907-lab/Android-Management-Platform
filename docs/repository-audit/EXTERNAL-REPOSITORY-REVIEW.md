# External Repository Review — TrojanCockroach and AdoBot

Review date: 2026-11-27  
Target product: Android Management Platform  
Review type: repository metadata, documentation, selected source/configuration inspection  
Execution status: **No third-party code, installers, APKs, EXEs, or scripts were executed. No source code was copied.**

## Executive decision

| Repository | Actual scope | Reuse decision |
|---|---|---|
| https://github.com/MinhasKamal/TrojanCockroach | Windows stealth Trojan/spyware | Reject entirely; no code or binary reuse |
| https://github.com/adonespitogo/AdoBot | Legacy Android spyware agent plus a separate server-side code area | Reject Android agent and spyware functions; independently reimplement only generic defensive patterns such as allowlist authorization, audit events, diagnostics validation and request logging |
| https://github.com/leephil1907-lab/Android-Management-Platform | Target commercial Android MDM product | Keep as the sole product repository |

## 1. TrojanCockroach audit

### Verified repository facts
- Default branch: `master`.
- Repository contains `README.md`, `LICENSE`, `TrojanCockroachStory.md`, C++ source files, Windows shortcut files and a bundled Windows executable.
- README identifies the project as a stealthy Trojan spyware for Windows.
- README describes credential and keystroke theft, email exfiltration, USB-drive propagation, disguised shortcut activation and startup persistence.
- The repository includes an MIT licence dated 2015.

### Assessment
This is not an Android management product, Android agent, remote-support implementation or legitimate device-enrollment framework. Its core behavior is incompatible with a transparent, customer-authorized MDM service.

### Decision
- Do not import source files, binaries, shortcuts or installation logic.
- Do not adapt persistence, stealth, credential capture, exfiltration or propagation behavior.
- The MIT licence does not change the security suitability decision. If the repository is ever discussed as a historical security example, keep that analysis isolated from production source.

## 2. AdoBot audit

### Verified repository facts
- Default branch: `main`.
- The Android source is under `android/`; its README identifies it as open-source Android spyware.
- The Android README documents hidden-app/stealth mode, remote command execution and scheduling, SMS and call-log collection, contact collection, location monitoring, SMS sending/forwarding, remote APK updates and persistent server-side data.
- The Android manifest requests location, SMS, contacts, call-log, phone-state, external-storage, boot-completed and outgoing-call permissions.
- The Android build uses compile SDK 26, target SDK 26, Android Support Library 26.1.0 and Room 1.0.0-beta2. These are old dependencies and an outdated Android build baseline.
- The Android changelog describes saving SMS and call logs locally while offline and syncing them later, including after the phone owner deletes those records.
- An MIT licence file exists at `android/MIT-License.txt`; no root-level LICENSE was found in the repository root listing. The licence scope for all server-side material and bundled third-party components is therefore not established by this inspection.
- The repository also contains a separate Python server area. Selected files include Telegram command authorization, audit event serialization, diagnostics validation, HTTP health/info endpoints and request logging.

### Assessment
The legacy Android agent is not a suitable base for a customer-facing MDM application. Its surveillance behavior, permission profile, outdated build configuration and device-owner privacy model conflict with this product's stated requirements.

The server-side patterns are generic engineering ideas, not a reason to adopt the spyware application:
- Explicit allowlist-based command authorization with fail-closed behavior.
- Positive allowlist of administrative user IDs.
- Structured audit events with accepted/denied/error outcomes.
- Diagnostics payload validation.
- Request logging and health/readiness endpoints.

### Decision
- Do not copy, fork into the product, compile, package, or deploy the AdoBot Android agent.
- Do not implement hidden-app mode, covert SMS triggers, background collection of personal communications, contact harvesting, stealth location collection or remote updates that bypass the user's management consent.
- Reimplement generic authorization, audit, diagnostics and observability patterns independently in the product's own codebase, with tenant/device scope and least privilege.
- Do not reuse AdoBot server code until each file's provenance, complete licence coverage and dependency notices have been independently established.

## 3. Capability comparison

| Product capability | TrojanCockroach | AdoBot Android agent | Proposed platform |
|---|---|---|---|
| Android enrollment / Device Owner | No | No documented MDM enrollment | Android Enterprise / supported provisioning |
| Device inventory and policy | No | Not a proper MDM policy engine | Headwind MDM adapter, subject to build/security validation |
| App deployment and managed configuration | No | Remote APK update feature, not an enterprise policy model | Managed app deployment and policy-controlled updates |
| Device health / diagnostics | No | Limited device-status reporting | Explicit, minimized diagnostics |
| Remote screen viewing | No | No documented legitimate support session | Separate visible, authorized remote-support service |
| Remote input | No | General remote commands, not a consent-based support session | Permission-gated interaction with session controls |
| File management | No | Not implemented in README; listed as future work | Explicitly scoped file actions and audit trail |
| Multi-customer tenancy | No | No verified production tenant isolation | Tenant-scoped API, database and device credentials |
| Customer accounts / billing | No | No verified commercial SaaS layer | Product-owned control plane and subscription service |
| Auditable operator actions | No | New server area has an audit-event pattern | Immutable, tenant-scoped audit trail |

## 4. Safe architecture recommendations

### Product-owned control plane
Build the commercial layer in this repository:
- Customer organizations and users.
- Roles and permissions.
- Device enrollment records and ownership/consent evidence.
- Tenant-scoped device inventory.
- Subscription plans, entitlements and usage metering.
- Remote-support session creation and approvals.
- Audit events and customer-visible activity history.
- Integration adapters to external MDM and remote-support services.

### Device management foundation
Continue evaluating:
- `h-mdm/hmdm-server` for Android MDM server capabilities.
- `h-mdm/hmdm-android` for the Android management agent.

Both repositories were previously identified as Apache-2.0 candidates. Before adoption, pin exact commits, inspect all transitive dependencies, build in CI and validate enrollment on real supported Android devices. Confirm the current licence and notices at the pinned revision.

### Remote support
Keep remote screen support as a separate subsystem. Headwind's published remote-access documentation describes a separate remote-support component using a mobile agent, server and browser client. The server-side project documentation points to an Android agent path, but the path `h-mdm/remote-control-android` returned 404 when checked in this review. Therefore the agent source, exact provenance, licence and compatibility remain **unverified**. Do not build against an assumed repository path or deploy an unverified binary.

A visible support session should require an explicit device-side approval or a clearly documented, customer-administered fully-managed-device policy. Provide session indication, operator identity, start/stop controls, short-lived credentials and a complete audit record. Screen capture and interaction must use supported Android mechanisms and respect OS/OEM limitations.

### Hybrid infrastructure
- Cloud: customer portal, authentication, tenant control plane, API, subscription and audit services.
- Dedicated Linux host(s): MDM server and remote media/signalling infrastructure where UDP/WebRTC/TURN networking is required.
- Database: PostgreSQL with tenant-scoped authorization and row-level isolation.
- Media: separate remote-support plane; do not route media streams through ordinary web API request handlers.
- Secrets: managed secret store; never commit device credentials, signing keys, tokens or production environment files.

## 5. Implementation priorities

1. Make the GitHub repository private before adding proprietary code, deployment configuration or secrets.
2. Establish a product threat model and data-minimization policy.
3. Define tenant, user, device, enrollment, policy, session and audit data models.
4. Implement authentication, MFA, role-based access and tenant isolation.
5. Define an MDM adapter contract and validate Headwind server/agent builds.
6. Verify Android Enterprise enrollment modes and OEM compatibility.
7. Locate and verify the remote-support agent source; do not assume the currently referenced path exists.
8. Implement support sessions with explicit approval, visible state, short-lived tokens and revocation.
9. Add file operations and app management only through explicit device policy and audit.
10. Add CI builds, dependency/SBOM scanning, secret scanning and reproducible release signing.

## 6. Verification status

| Check | Status |
|---|---|
| TrojanCockroach README, tree and MIT licence inspected | Complete |
| AdoBot Android README, manifest, Gradle config and changelog inspected | Complete |
| AdoBot selected server authorization/audit/diagnostics modules inspected | Complete |
| AdoBot complete dependency and licence audit | Not complete |
| Any third-party code executed | No |
| Any third-party code copied into product | No |
| Headwind MDM server/agent clean builds | Not run |
| Remote-support Android agent identity and licence | Unverified |
| Product implementation or deployment | Not started; repository remains documentation-only |

## Final decision

Neither TrojanCockroach nor AdoBot should become a code dependency of the commercial Android Management Platform. Keep the platform's product code independent. Use only independently implemented, ordinary defensive engineering patterns from the AdoBot server review. Continue with Headwind MDM as the leading MDM candidate, and keep remote support separate until its exact Android agent source, licence, build and supported Android behavior are verified.
