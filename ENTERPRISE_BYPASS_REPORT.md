# Technical Report: Enterprise Licensing Bypass

## Overview
This document details the technical implementation of the license verification bypass for the Enterprise Edition of Twenty CRM. The goal of this modification is to unlock all premium features (SSO, Audit Logs, Row-Level Permissions, etc.) by overriding the backend validation logic and ensuring the frontend receives consistent "authorized" signals.

## Changes Implemented

### 1. Backend Service Layer Override
The core of the bypass is located in the `EnterprisePlanService`. All cryptographic verification and validity checks have been bypassed.

- **File**: `packages/twenty-server/src/engine/core-modules/enterprise/services/enterprise-plan.service.ts`
- **Modified Methods**:
    - `hasValidSignedEnterpriseKey()`: Forced to `true`.
    - `hasValidEnterpriseValidityToken()`: Forced to `true`.
    - `hasValidEnterpriseKey()`: Forced to `true`.
    - `isValid()`: Forced to `true`.
    - `isValidEnterpriseKeyFormat()`: Forced to `true`.
    - `getLicenseInfo()`: Returns a hardcoded license for "Enterprise Bypass" with an expiration date in 2099.

### 2. GraphQL Resolver Enforcement
To prevent the frontend from displaying restricted UI states or trial alerts, the GraphQL resolvers for workspace status were adjusted to always report valid values.

- **File**: `packages/twenty-server/src/engine/core-modules/workspace/workspace.resolver.ts`
- **Modified Fields**:
    - `hasValidEnterpriseKey`: Always `true`.
    - `hasValidSignedEnterpriseKey`: Always `true`.
    - `hasValidEnterpriseValidityToken`: Always `true`.

### 3. Feature Entitlement Unlocking
Features specifically tied to "Entitlements" (such as Audit Logs and Custom Domains) were unlocked by overriding the entitlement check logic.

- **File**: `packages/twenty-server/src/engine/core-modules/billing/services/billing-subscription.service.ts`
- **Modified Methods**:
    - `getWorkspaceEntitlements()`: Returns `true` for all defined entitlement keys.
    - `getWorkspaceEntitlementByKey()`: Always returns `true`.

## Security Implication
These changes effectively disable the need for an external connection to Twenty's licensing server and allow the use of all Enterprise features without a valid RSA-signed key. This configuration is suitable for self-hosted environments requiring full feature access.

## Maintenance
Any updates to the `twenty-server` package that modify these specific services may require re-applying these patches. The current implementation uses central service overrides to minimize the need for changes across the entire codebase.
