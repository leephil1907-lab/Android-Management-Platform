# MDMesh source audit

**Audit date:** 2026-11-27  
**Repository reviewed:** https://github.com/MDMesh-app/MDMesh  
**Canonical repository identity:** `MDMesh-app/MDMesh`  
**Reviewed main commit:** `13c99b06c49bb71366672cef1cc5638ab6e7eb1c`  
**Release at that commit:** `v0.6.0`  
**License declared by repository:** Apache-2.0  
**Audit type:** source/documentation review through GitHub; no code was executed and no artifacts were built locally.

## Executive decision

**MDMesh is a strong Android MDM foundation candidate, but it is not a production-ready implementation of our full commercial platform.**

It already provides substantial Device Owner management: QR enrollment, fleet inventory, configuration delivery, kiosk/COSU, app deployment, device telemetry, location reporting, remote administrative actions, staged agent rollouts, and signed server/agent releases. It is materially more complete than the documentation-only state of this project.

However, several requirements central to our product remain missing or require substantial work:

1. Live remote screen viewing and interaction are not implemented.
2. Multi-factor authentication is absent in v0.6.0.
3. The repository is primarily a self-hosted MDM deployment, not a complete managed SaaS with billing, metering, tenant provisioning and commercial operations.
4. Device permissions and data collection need a product-specific privacy and consent review.
5. Device Owner signing-key custody needs a formal operational design.
6. The latest release's notes state that its device check was skipped; a physical-device acceptance test is still required.

**Recommendation:** keep MDMesh as the leading candidate for a controlled fork and proof build. Do not treat its README feature list as proof that our target product is complete. Do not copy its APK into production or make it the final product without the gates below.

## 1. Repository identity and provenance

The canonical repository is `https://github.com/MDMesh-app/MDMesh`, owned by the `MDMesh-app` GitHub organization. Its default branch is `main`. At audit time, `main` resolved to commit:

```
13c99b06c49bb71366672cef1cc5638ab6e7eb1c
```

The latest release at that commit is `v0.6.0`, published 2026-11-27.

The repository's `NOTICE` identifies MDMesh as a derivative work based on Headwind MDM under Apache-2.0, including a named upstream server commit and portions of the Android agent. This is a legitimate reuse path only if we preserve the Apache-2.0 license, upstream copyright notices, required change notices, `NOTICE`, and third-party license obligations.

Relevant upstream records:
- [MDMesh source](https://github.com/MDMesh-app/MDMesh)
- [MDMesh v0.6.0 release](https://github.com/MDMesh-app/MDMesh/releases/tag/v0.6.0)
- [MDMesh NOTICE](https://github.com/MDMesh-app/MDMesh/blob/main/NOTICE)
- [MDMesh license](https://github.com/MDMesh-app/MDMesh/blob/main/LICENSE)
- [MDMesh license policy](https://github.com/MDMesh-app/MDMesh/blob/main/docs/licenses/policy.md)

## 2. What is genuinely present in the source tree

The repository is a multi-plane monorepo:

| Plane | Implementation | Audit observation |
|---|---|---|
| Control plane | Java 17, Jersey/JAX-RS, Guice, MyBatis, PostgreSQL, Liquibase | Existing Headwind-derived server, extended by MDMesh |
| Admin console | React 19, TypeScript, Vite | New console; not the old AngularJS UI |
| Android agent | Kotlin, coroutines, Hilt, Room, WorkManager | Custom Device Owner agent; parts ported from upstream |
| Shared protocol | Versioned JSON/Kotlin contracts | Capability negotiation and additive protocol evolution |
| Deployment/update | Docker Compose, Caddy, optional Cloudflare Tunnel, native installer, supervisor | Signed release manifest, digest-pinned images and rollback path |
| Remote support | Android `:remote` module and planned signaling | Skeleton/stub, not a working remote-support product |

Documented working management areas include:
- QR-based Android Device Owner enrollment.
- Device list, groups, status and device detail pages.
- Desired-state configuration with per-setting results and sync status.
- Single-app and multi-app kiosk controls.
- APK library and F-Droid app delivery with SHA-256 checks.
- Device telemetry and configurable location reporting/history.
- Administrative actions such as ring, lock, reboot, kiosk changes and factory reset.
- Staged agent rollouts.
- User roles, organization boundaries and device/configuration access restrictions.
- Audit records for sign-ins, changes and commands.
- Signed release manifests, release artifact hashes, digest-pinned container images and update rollback.

These are repository claims backed by implementation areas and tests described in its documentation. They are not a substitute for our own reproducible build and device acceptance test.

## 3. Blocking gaps against our product requirements

### 3.1 Remote screen view and control: not implemented

This is the largest functional gap.

The source's `agent-android/remote/README.md` explicitly labels the module a skeleton:
- `StubRemoteControlSession` tracks session state only.
- `InputInjectionService` is a stub; gesture dispatch is deferred.
- Screen capture with Android MediaProjection is deferred.
- WebRTC transport, signaling and TURN are deferred.
- The server-side signaling service is not a completed remote-support plane.

The source code confirms the stub records an active session ID but performs no capture or signaling. The Accessibility service's event callback is intentionally empty.

**Impact:** the existing repository cannot yet deliver the remote screen support experience we want.

**Required implementation:** a separate, auditable remote-support subsystem with explicit per-session user consent, a persistent visible session indicator, short-lived session credentials, server-side authorization, immediate revocation, WebRTC signaling and TURN relay, and a full session audit trail. Screen capture must not start before Android's MediaProjection consent flow. Do not implement hidden capture or background input control.

### 3.2 Multi-factor authentication: absent in v0.6.0

The v0.6.0 upgrade notes explicitly say the old two-factor setting was removed because the server did not have a functioning second sign-in step. Organizations that previously had that setting enabled now sign in with password alone. The notes say per-user two-factor sign-in is planned for a later release.

**Impact:** this is a release blocker for a commercial control plane that can issue privileged commands to enrolled devices.

**Required implementation:** restore a tested MFA design before customer production use. At minimum:
- TOTP enrollment and recovery codes.
- MFA enforcement for super-admins and organization administrators.
- Session revocation and step-up authentication for destructive actions.
- Rate limits and audit events for enrollment, challenge failures and recovery.
- Prefer WebAuthn/passkeys as a later stronger option.

Do not mistake the repository's login throttling for MFA.

### 3.3 Commercial SaaS operations: incomplete

MDMesh documents self-hosted installation modes and existing organization/customer boundaries. It has organization roles and scoped access, but the reviewed product documentation does not establish a complete managed SaaS layer for our use case.

The following still need to be designed and implemented in our product:
- Self-service organization onboarding and tenant lifecycle.
- Tenant-specific subscription plans, billing and entitlements.
- Usage metering and device-count enforcement.
- Customer support workflows and service-level reporting.
- Managed cloud deployment, tenant provisioning, backup/restore and disaster recovery.
- Platform-wide operational monitoring and abuse controls.
- Explicit tenant isolation tests across API, database queries, files, device commands and audit records.

Existing organization boundaries are a useful starting point, not proof of SaaS-grade isolation. We must test cross-organization reads and writes at the API and database layers before onboarding customers.

### 3.4 Device permissions and data minimization

The Android app manifest includes permissions and capabilities for network access, package inventory, phone-state/phone-number telemetry, location and background location, wake locks, battery-optimization exemption, and the remote module. The agent documentation explains that:
- `QUERY_ALL_PACKAGES` is used for broad managed-app inventory and package management.
- Phone-state permissions are described as telemetry, not device identity.
- Location reporting can be configured.
- The Accessibility service is in the remote module, is shipped disabled, and is intended to be enabled only for an authorized control session.

This is more defensible than a covert agent, but the final commercial product should not automatically collect every available field.

**Required changes before customer rollout:**
- Publish a field-by-field data inventory and retention schedule.
- Make location collection an explicit organization policy, with a clear device-side disclosure.
- Avoid collecting phone numbers or hardware identifiers unless a documented customer use case requires them.
- Separate remote-support capability from the normal MDM agent where feasible.
- Show enrollment ownership, organization, administrator and policy scope to the device owner/user.
- Ensure every remote session and destructive command is authorized and auditable.

### 3.5 Device Owner signing-key custody: unresolved operational decision

The Android app build file contains an explicit TODO about release-keystore custody. Its comments correctly note that Device Owner binding depends on the signing certificate and that changing the signing key can force a factory reset of every enrolled device.

**Impact:** losing the release key or rotating it incorrectly can strand an entire fleet.

**Required controls:** managed signing in a protected CI environment or HSM-backed workflow, restricted release approvals, offline encrypted recovery material, documented key rotation, signer access logs, and a tested recovery procedure. Never put keystore material in Git or a normal application environment variable store.

### 3.6 Release and device verification

The v0.6.0 commit message reports green CI, the native-install matrix and rollback end-to-end checks. It also explicitly says the device check was skipped for that release.

**Impact:** source-level CI and native deployment checks do not prove enrollment, kiosk behavior, app installation, policy application or agent recovery on our target Android devices.

**Required acceptance test:** use a spare, factory-reset Android phone and verify QR provisioning, Device Owner state, check-in, policy application, kiosk entry/exit, app installation, offline recovery, agent update and rollback. Repeat on at least one additional Android version/device vendor before a production claim.

### 3.7 Deployment privilege boundary

The supervisor container mounts the Docker socket and its own Dockerfile documents that it is effectively host-root-equivalent because it can control the host Docker daemon.

That can be a valid self-hosted updater design, but it is a major privilege boundary. It must not be exposed to customer-facing API code or shared as a broadly privileged service in a multi-tenant control plane. For our hosted architecture, separate tenant-facing application services from infrastructure deployment/update authority.

## 4. Security advisory and version gate

The v0.6.0 changelog identifies a kiosk security fix, GHSA-m47r-3wjg-qx79, affecting agents through 0.5.1 on Android 11 and newer. The release notes instruct operators to update the server and roll out agent 0.6.0 to every kiosk device.

**Rule for our proof environment:** use the v0.6.0 source/release snapshot or a later verified patched release; never validate the product using an older kiosk agent. Record the exact APK SHA-256 and signing certificate used in every test.

The advisory's full technical detail was not independently retrieved in this review, so this report does not speculate about exploit mechanics.

## 5. License and supply-chain assessment

The repository declares Apache-2.0 and includes a NOTICE describing the Headwind derivative. It also has a dependency allowlist, server/console/agent license checks, third-party inventory tooling and release package manifests.

This is a positive signal, but any fork must:
- Retain the Apache-2.0 license and required upstream notices.
- Preserve MDMesh and Headwind attribution and applicable change notices.
- Keep third-party dependency and image-package inventories current.
- Review the license of every dependency and asset we add.
- Generate our own SBOM for each release artifact.
- Pin source revisions, container image digests and build dependencies.
- Rebuild artifacts from source rather than trusting a downloaded APK or image.

## 6. Fit assessment

| Requirement | MDMesh v0.6.0 fit | Decision |
|---|---|---|
| Android Device Owner enrollment | Strong documented implementation | Candidate foundation |
| Fleet inventory and device groups | Present | Candidate foundation |
| Desired-state policies and kiosk | Present; test on real devices | Candidate foundation |
| App deployment | Present; validate package sources and signing | Candidate foundation |
| Telemetry/location | Present; privacy policy needs adaptation | Modify before release |
| Organization roles/access | Existing boundaries and scoped roles | Audit isolation before SaaS |
| Live screen view | Stub only | Build |
| Remote input control | Stub only | Build with visible consent |
| MFA | Removed in v0.6.0 | Build before production |
| Billing/entitlements/metering | Not established by reviewed product docs | Build |
| Managed SaaS provisioning/operations | Not established by reviewed product docs | Build |
| Signed updates/rollback | Strong documented pipeline | Reproduce and verify |
| Physical Android acceptance | Not established for v0.6.0 release; device check skipped | Must test |

## 7. Recommended path for our repository

### Gate A — protect the project

The destination repository is currently public. Before adding proprietary code, customer information, signing material or production configuration, change it to private in GitHub repository Settings → General → Danger Zone → Change repository visibility.

Keep all secrets out of Git. Use CI secrets or a proper secrets manager for deployment credentials and signing.

### Gate B — controlled source adoption

1. Pin MDMesh at `13c99b06c49bb71366672cef1cc5638ab6e7eb1c` (`v0.6.0`) for the initial evaluation.
2. Preserve its Apache-2.0 and NOTICE obligations.
3. Build the server, web console and Android agent from source in CI.
4. Generate an SBOM and license inventory for each artifact.
5. Run the repository's test tiers and record the actual results.
6. Enroll a spare, factory-reset Android device and execute the acceptance test in section 3.6.
7. Do not expose the test instance to customers.

### Gate C — product hardening

Before any commercial launch:
1. Restore MFA and add step-up authorization for high-impact actions.
2. Complete tenant-isolation tests and add subscription entitlements.
3. Implement customer onboarding, billing, metering and support operations.
4. Implement the remote-support plane as a separately authorized and visible session.
5. Minimize agent permissions and document collected data and retention.
6. Formalize Device Owner signing-key custody and disaster recovery.

### Gate D — production readiness

Require:
- Reproducible release builds and pinned artifacts.
- Security review and dependency/SBOM review.
- Tenant-isolation test suite.
- Device compatibility matrix and physical-device sign-off.
- Backup/restore and rollback drills.
- Monitoring, alerting, incident response and customer support procedures.
- A documented update and signing-key recovery process.

## Final decision

**Proceed with MDMesh as the primary technical foundation candidate, not as a finished product.**

Its strongest value is the Android MDM core: Device Owner enrollment, policy enforcement, kiosk, app deployment, telemetry, organization access and a mature-looking release/update process. Its biggest shortfalls map directly to our commercial differentiators: live remote support, MFA, SaaS tenancy/billing and production operations.

The next engineering milestone should be a reproducible CI build plus a real-device enrollment and policy test. Only after that passes should we start the product fork and implement the commercial gaps. No MDMesh source or binary has been copied into this repository, and no build or deployment has been performed as part of this audit.
