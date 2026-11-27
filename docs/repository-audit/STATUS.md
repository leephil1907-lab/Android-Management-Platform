# Repository Audit — Initial Findings

Audit date: 2026-11-27  
Status: **In progress — source inspection only**  
No upstream code has been copied, patched, executed or deployed as part of this audit.

## 1. Candidate repositories

| Component | Source | Initial finding | Decision |
|---|---|---|---|
| MDM server | https://github.com/h-mdm/hmdm-server | Public repository; default branch `master`; Maven multi-module Java project; Apache-2.0 LICENSE present | Keep as primary MDM candidate; build and security review required |
| Android MDM agent | https://github.com/h-mdm/hmdm-android | Public repository; default branch `master`; Gradle Android project; Apache-2.0 LICENSE present | Keep as primary enrollment/management candidate |
| Remote-support server candidate | https://github.com/met989/headwind_remote_control | Third-party repository, not under the h-mdm owner; README describes a custom Headwind/Janus integration and claims Apache-2.0 | Unverified candidate only; inspect complete source, provenance, dependencies and license notices before use |
| Remote Android agent | No verified official repository identified yet | Earlier guessed `h-mdm/remote-control-android` path returned GitHub 404 | Do not depend on this path; locate and verify the actual agent source |

## 2. Findings from source inspection

### Headwind MDM server

- README describes QR-code enrollment, application deployment, device groups, configurations, device status and plugin extensibility.
- `pom.xml` defines a multi-module Maven build.
- The POM declares Java 8 source compatibility and includes older framework/dependency versions. This requires dependency vulnerability review and a reproducible build on a supported JDK.
- The repository contains an Apache License 2.0 file. Preserve copyright and NOTICE material when applicable.

### Headwind Android agent

- README documents Android Studio and command-line Gradle builds.
- Current checked-in Gradle wrapper points to Gradle 8.13.
- Root build file declares Android Gradle Plugin 8.13.2.
- README's older instruction to install Gradle plugin 5.1.1 conflicts with the checked-in wrapper/build configuration; the wrapper and current build files should be treated as the source of truth, then validated in CI.
- README describes optional Device Owner setup using Android Debug Bridge. Production enrollment must use the supported, documented provisioning flow.
- Apache License 2.0 file is present.

### Third-party remote-support candidate

The repository README claims:
- Janus/WebRTC screen streaming and browser viewing
- Android MediaProjection screen capture
- Accessibility-based user interaction
- Headwind MDM SSO integration
- An installer that configures Janus, Tomcat, web assets and firewall rules

The inspected installer is a privileged system installer. It installs operating-system packages and modifies server configuration. **Do not run it on production or development infrastructure during the audit.** Review every script and packaged artifact in a disposable test VM first.

The repository's README and LICENSE are claims by that repository; they do not establish upstream endorsement, code provenance, ownership of all bundled assets, or commercial rights to any separately sourced component.

## 3. Immediate blockers

1. Identify the actual Android remote-support agent source and verify its relationship to the server plugin.
2. Inspect all installer, update and uninstall scripts before execution.
3. Inspect bundled WAR/APK/JAR files and determine whether reproducible source builds are available.
4. Record exact commit SHAs and tags for every dependency.
5. Run dependency and secret scans.
6. Build the MDM server and Android agent in clean environments.
7. Verify Android version, OEM and Device Owner enrollment compatibility.
8. Complete license and commercial redistribution review for every component.

## 4. Build verification matrix

| Project | Required check | Result |
|---|---|---|
| hmdm-server | Maven clean package in isolated environment | Not run |
| hmdm-android | Gradle wrapper build in isolated environment | Not run |
| Remote-support server candidate | Static review, dependency scan, script review, reproducible build | Not run |
| Remote-support Android agent | Source identity, permissions review, build and device test | Not run |

## 5. Acceptance criteria for closing Milestone 1

- Every upstream source has a verified repository URL, commit SHA, tag and license.
- No unverified or unavailable repository path remains in the dependency plan.
- Clean build results and toolchain versions are recorded.
- Third-party binaries have traceable origins or are excluded.
- Privileged scripts have been reviewed without running them on a live host.
- A written decision identifies which components are adopted, replaced or rejected.

## 6. Next milestone

After the audit closes: create pinned integration references, CI build checks, a development-only Compose environment and the first MDM adapter contract. Do not add production secrets or deploy customer devices at this stage.


## 7. External repository review — 2026-11-27

Detailed review: [EXTERNAL-REPOSITORY-REVIEW.md](./EXTERNAL-REPOSITORY-REVIEW.md)

- `MinhasKamal/TrojanCockroach`: confirmed Windows stealth Trojan/spyware. Rejected entirely; no source or binary reuse.
- `adonespitogo/AdoBot`: Android agent confirmed as spyware with hidden-app mode and sensitive personal-data collection. Rejected as an agent. Generic server-side authorization, audit-event, diagnostics and request-logging patterns may inform independent implementations only.
- AdoBot's Android folder contains an MIT licence file, but complete repository/server/dependency licence scope is not established.
- The published Headwind Remote documentation references an Android agent path that returned 404 when checked. Exact source identity and licensing remain unresolved; do not depend on that path or an unverified binary.
- No code from either external repository has been copied or executed.
- No application code has been implemented, built or deployed in this repository; it remains documentation-only.


## 8. MDMesh canonical-source audit — 2026-11-27

Detailed report: [MDMesh source audit](./MDMESH-SOURCE-AUDIT.md).

- Canonical repository verified as `MDMesh-app/MDMesh`; the older `Likhith2007/mdm` identity is an alias/older repository identity, not the canonical URL to pin.
- Reviewed `main` at `13c99b06c49bb71366672cef1cc5638ab6e7eb1c`, corresponding to v0.6.0.
- MDMesh is a strong Android MDM foundation candidate: Device Owner QR enrollment, policies, kiosk, app delivery, telemetry, organization roles, audit, signed releases and updater/rollback.
- Live remote screen capture, WebRTC signaling and input injection remain stubs/deferred.
- v0.6.0 removes the unused two-factor setting; the upgrade notes state users authenticate with passwords alone until MFA is implemented.
- The release changelog identifies a kiosk security fix affecting agents through 0.5.1 on Android 11+; use 0.6.0 or later and update every kiosk agent.
- The v0.6.0 commit reports CI/native tests green but explicitly says the device check was skipped.
- Release signing-key custody is called out as an unresolved operational TODO in the Android build configuration.
- License is Apache-2.0 with Headwind-derived code; preserve NOTICE, attribution, change notices and dependency inventory.
- No MDMesh source or binary has been copied, executed, built or deployed in this repository.

## 9. Updated decision

MDMesh replaces the earlier standalone Headwind repositories as the leading candidate for a controlled foundation evaluation. It is not approved for customer production until a clean CI build, real-device enrollment/policy test, MFA restoration, tenant-isolation verification, signing-key custody plan and remote-support implementation are complete.
