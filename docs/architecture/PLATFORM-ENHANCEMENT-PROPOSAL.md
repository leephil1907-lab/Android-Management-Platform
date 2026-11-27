# Platform Enhancement Proposal — Modern Android Fleet Management

Date: 2026-11-27
Status: Proposal; no implementation or third-party code import has occurred.

## Product direction

Evolve Android Management Platform from a basic MDM wrapper into a commercial, multi-tenant Android fleet-management and authorized remote-support platform.

Product principles:
- Android-first. Do not dilute the first release with desktop/iOS management.
- Transparent device enrollment and clearly communicated management scope.
- Use supported Android Enterprise / Device Owner provisioning and documented device APIs.
- Separate the commercial control plane from device-management and remote-support services.
- Tenant isolation, explicit authorization, auditable actions, and secure lifecycle management are core requirements.
- Avoid vendor lock-in by implementing adapters and a versioned device-agent contract.

## Sophisticated project candidate discovered

### MDMesh
Repository currently inspected: https://github.com/Likhith2007/mdm
README identifies it as MDMesh and links the project identity https://github.com/MDMesh-app/MDMesh. This project claims an Android-focused architecture with:
- Java server, React/TypeScript console, Kotlin Device Owner agent.
- PostgreSQL backend.
- Versioned, capability-negotiated agent protocol.
- QR enrollment.
- Kiosk/lock-task mode.
- Managed APK and F-Droid catalog/deployment.
- Device status and telemetry.
- Signed releases and update verification.
- Staged canary rollout and rollback.
- Docker deployment and edge/TLS setup.

The repository README claims Apache-2.0. The inspected LICENSE file is Apache-2.0. These are preliminary repository-level findings only. Verify the canonical repo, exact commit, release tags, build workflows, source history, dependencies, security reports, and licensing of all bundled assets before adoption. The GitHub repository inspected is public and read-only to this account. No code has been copied or run.

### FleetDM
Repository: https://github.com/fleetdm/fleet
Fleet offers a more mature cross-platform fleet/security-management architecture, APIs, GitOps, webhooks, and observability. It supports Android among its listed platforms, but is not an Android-only management agent foundation. Its repository has license boundaries: the free core is MIT, while `ee/` and other components can have separate licensing. Do not import Enterprise Edition code or assume the entire repository is under MIT. Use it as an architectural reference for API, audit, GitOps, and security posture, not as the first Android agent dependency.

## Candidate comparison

| Candidate | Android depth | Modern agent stack | Self-hosting | Commercial product fit | Decision |
|---|---|---|---|---|
| Headwind MDM | Android-focused | Existing agent; compatibility/build review needed | Yes | Useful baseline / adapter candidate | Continue validating |
| MDMesh | Android-focused, claims from-scratch Kotlin Device Owner agent | Kotlin + capability protocol | Yes, Docker/native paths claimed | Closest apparent technical match, but early verification required | Audit and pilot; do not blindly fork |
| FleetDM | Multi-platform device/security fleet | Broad ecosystem; Android support must be tested against desired features | Yes | Useful reference; less focused on Android support operations | Architecture reference |
| Commercial MDMs (Scalefusion, SureMDM, Hexnode, etc.) | Android enterprise feature depth | Vendor-controlled | Cloud/on-prem varies | Benchmark features and UX; not code reuse | Product benchmark only |

## Proposed product architecture: five planes

### 1. Tenant/control plane
- Organizations, customer account lifecycle, roles, teams, delegated administrators.
- Device ownership/enrollment records and consent/policy records.
- Plans, billing, feature entitlements, limits and usage.
- Tenant-specific branding and support settings.
- Provisioning and lifecycle management of isolated MDM backends.
- API keys and integrations, with rotation and scoped permissions.

### 2. Device management plane
- Enrollment token and QR lifecycle.
- Device groups, labels, policies and configuration templates.
- Application catalogue, private app distribution, release channels and staged rollouts.
- Kiosk profiles, managed configurations and permitted device actions.
- Device inventory, last check-in, OS/build, battery, storage and network health.
- Offline command queue with expiry, deduplication and idempotency.
- Device-agent capability negotiation and backward-compatible API.

### 3. Authorized remote support plane
- Support request, operator identity, device approval/policy and session scope.
- Visible active-session indicator and local stop/deny controls wherever OS allows.
- Screen-sharing session with short-lived credentials.
- Permission-gated interaction, not arbitrary shell or covert control.
- Session timeout, revoke, reconnect, operator audit and metadata retention.
- Separate signaling/media infrastructure (WebRTC/TURN) from REST control plane.
- No recording by default; if offered later, explicit disclosure, authorization and retention controls.

### 4. Security and reliability plane
- MFA for administrators; RBAC/ABAC for sensitive actions.
- Per-device revocable credentials, TLS, rotation and replay protection.
- Tenant isolation tests and authorization tests for every device operation.
- Audit events: actor, tenant, target device, action, reason, timestamp, outcome, correlation ID.
- Signed Android-agent releases; signature/hash verification; staged rollout, health gates and rollback.
- Backups, recovery runbooks, migration safety and rate limits.
- SBOM, dependency and secret scans, SAST, image scanning, CI build provenance.

### 5. Operations and analytics plane
- Fleet health, enrollment funnels, check-in reliability, policy compliance.
- Alert rules (offline, low battery, low storage, failed rollout, compliance drift).
- Support queue, ticket linking, operator workload and SLA.
- Exportable audit and reports.
- Metrics/logs/traces with privacy-minimizing collection.

## Proposed first-party repository layout

```
apps/
  web/                       # React + TypeScript console
  api/                       # commercial control-plane API
services/
  tenant-control/
  enrollment/
  device-events/
  policy-engine/
  app-catalog/
  remote-support-signaling/
  billing/
agents/
  android/                   # Kotlin, Device Owner, transparent agent
packages/
  api-contracts/
  ui/
  sdk/
infra/
  docker/
  terraform/
  monitoring/
docs/
  architecture/
  security/
  enrollment/
  operations/
  decisions/
```

This is a target layout, not an instruction to prematurely split into microservices. Start with a modular monolith plus independently deployed MDM/support components; split services only when deployment or scaling boundaries justify it.

## Proposed release stages

### Stage 0 — Candidate audit and proof-of-build
- Pin exact upstream commits.
- Validate canonical MDMesh repository identity and project activity.
- Build MDMesh server, web console and Android agent from source in isolated CI.
- Inspect Docker/native install/update scripts; do not run privileged scripts on production.
- Review Apache-2.0 compliance, bundled assets, dependencies, SBOM and security issues.
- Test QR/Device Owner enrollment on a reset spare device and supported Android versions.
- Test remote actions and update rollback in a disposable environment.
- Compare MDMesh with Headwind and define an adapter/ownership decision.
Exit: reproducible builds, clean pilot enrollment, documented limitations, source and licence inventory.

### Stage 1 — Commercial control-plane MVP
- Login, MFA, organizations, roles and tenant isolation.
- Dashboard, device inventory, search/filter, device detail.
- Enrollment flow and revoke/decommission.
- Device groups and policy templates.
- Audit log and basic alerting.
- MDM adapter integration with isolated customer tenancy.
Exit: tests prove one tenant cannot enumerate or control another tenant's devices.

### Stage 2 — Enterprise device operations
- App catalogue, private APK distribution, staged release channels.
- Kiosk configuration, policy templates, compliance status.
- Offline command queue, idempotency and capability negotiation.
- Device health and diagnostic reports.
- Device lifecycle: enroll, transfer, suspend, lost, wipe, retire.

### Stage 3 — Remote-support pilot
- Device-initiated/approved session flow.
- Session broker and WebRTC signaling.
- Visible consent, short-lived session credentials, revocation.
- Screen share and supported user-authorized interaction.
- Session audit, operator permissions and reliability metrics.
Exit: successful tests across selected OEMs/Android versions; no hidden or unattended access outside explicit managed-device policy.

### Stage 4 — Commercial SaaS
- Subscription, billing, entitlements, usage quotas.
- Customer onboarding and self-service enrollment.
- Tenant-specific branding, support and reports.
- Backup/restore, disaster recovery, incident procedures.
- Production monitoring, release channels, support and SLA.

## Benchmark features to study (do not copy vendor UI)
- Modern device-detail page with activity timeline and action status.
- Fleet filters, saved views, bulk operations with preview and confirmation.
- Policy-as-code / versioned policy templates.
- Canary deployment, rollout progress, health gates and automatic rollback.
- Device agent capabilities and protocol compatibility.
- Recovery UI that remains available if primary update fails.
- Tenant provisioning and cost/usage dashboard.

## Non-goals / prohibited product behaviors
- Hidden app or covert enrollment.
- Secret collection of SMS, contacts, call logs, credentials, keystrokes, camera or microphone.
- Remote activation of screen capture without the applicable consent/policy.
- Self-propagation, deceptive installers, or persistence that evades device owners.
- Unrestricted arbitrary shell commands.
- Bypassing Android security, consent prompts or app-store protections.

## Next concrete action

Create a clean CI proof-of-build branch or pipeline for the chosen candidate. Before importing any code, run the Stage 0 audit and record exact commits and test results. Prefer a thin adapter and first-party commercial layer if upstream is healthy; otherwise implement the first-party Android agent and protocol incrementally.
