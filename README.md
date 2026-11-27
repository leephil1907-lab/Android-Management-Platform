# Android Management Platform

A commercial, hybrid-hosted Android device-management platform for organizations and explicitly enrolled personal or company-owned devices.

## Product scope

- Organization and team management
- QR-based Android enrollment and device inventory
- Device status, policy and application management
- Authorized remote-support sessions with visible device-side indicators
- Permission-controlled file operations and diagnostics
- Tenant isolation, audit history and subscription management

This project is an integration platform. Upstream MDM and remote-support components remain independently versioned and are not copied into this repository until their build, security, compatibility and licensing reviews are complete.

## Architecture

- **Web:** React + TypeScript
- **Control API:** modular backend services
- **Identity:** Supabase Auth or a reviewed equivalent
- **Data:** PostgreSQL
- **MDM:** Headwind MDM adapter
- **Remote support:** separately deployed WebRTC/Janus service, subject to source and license verification
- **Infrastructure:** cloud control plane plus dedicated Linux VPS nodes for MDM and real-time media

## Repository layout

```text
apps/                 Web dashboard and customer/admin portals
services/             API, tenancy, device, MDM, session, billing and audit services
android/              First-party Android agent work
integrations/          Adapters for reviewed upstream components
infrastructure/        Cloud, VPS, Docker and monitoring configuration
docs/                  Architecture, audits, security and deployment decisions
scripts/               Development and verification scripts
```

## Current status

**Milestone 1 — repository audit.** No production services or Android agent have been implemented or deployed yet. See [Repository audit status](docs/repository-audit/STATUS.md).

## Development principles

1. Enroll devices through documented, authorized Android Enterprise flows.
2. Keep remote-support sessions visible and explicitly authorized.
3. Enforce tenant boundaries in the backend, not only in the UI.
4. Never expose upstream MDM administration credentials to browser clients.
5. Pin upstream versions and record source commit SHAs.
6. Preserve upstream copyright, notices and license obligations.
7. Do not publish secrets, signing keys, service-account credentials or customer data.

## Local development

Toolchain and exact build commands will be finalized after the upstream baseline audit. Use the upstream repositories' own build instructions until then.

## License

The license for this integration project is intentionally not selected yet. Review all upstream licenses, notices, trademarks and commercial redistribution terms before choosing a project-wide license.
