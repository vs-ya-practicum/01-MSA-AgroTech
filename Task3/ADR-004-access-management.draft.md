# ADR-004: Packaged Access Management (Draft)

## Status

Draft

## Decision

Use self-hosted ZITADEL OSS as one packaged component for Authentication and Authorization (RBAC) in both architecture variants.

## Justification

- Provides OIDC/OAuth2, MFA, and role-based access control.
- Supports organization and tenant concepts for future SaaS evolution.
- Has no subscription fee in the OSS edition.
- Keeps identity data and operations under company control.
- Avoids custom implementation of core identity functions.

## Rejected alternatives

- Custom Authentication and Authorization: higher security and maintenance risk.
- Keycloak: capable, but heavier operationally for this MVP.
- Authentik: viable, but less aligned with the planned SaaS organization model.
- ZITADEL Cloud: adds recurring vendor fees and external service dependency.
