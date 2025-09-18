---
title: "API Page Permission Model"
domain: "seth-security"
difficulty: "intermediate"
bc_versions: "14+"
tags: ["api-permissions", "access-control", "security"]
samples: "samples/api-permission-model.md"
---
# API Page Permission Model

## Overview

API Pages in Business Central implement a comprehensive permission model that controls access to data and operations through OData endpoints. Understanding this permission model is essential for implementing secure API solutions that properly protect business data.

The permission system operates at multiple levels including page access, table permissions, field-level security, and operation-specific restrictions. Each level contributes to the overall security posture of API implementations.

Proper permission configuration ensures API consumers can only access authorized data and perform permitted operations, while maintaining system security and compliance requirements.

## Page-Level Access Control

API Page access requires explicit permissions through the standard BC permission system. Users or service accounts accessing API endpoints must have appropriate permissions assigned through permission sets.

Page permissions control whether an API endpoint is accessible to a particular user or service account. Without proper page permissions, API requests will be rejected with authentication errors.

Consider the principle of least privilege when assigning page permissions. Grant only the minimum access required for legitimate API operations to reduce security exposure.

## Table Permission Requirements

API Pages inherit table permissions from their source tables, requiring appropriate read, insert, modify, or delete permissions based on supported operations. Table permissions directly control what data operations are available through the API.

Different permission levels enable different OData operations. Read permissions enable GET operations, while insert permissions enable POST operations for creating new records through the API.

Table permissions may be filtered or restricted based on field values, enabling row-level security implementations. This allows fine-grained control over which records are accessible through API endpoints.

## Field-Level Security

Individual field permissions control which table fields are accessible through API Pages. Fields without appropriate permissions are automatically excluded from API responses and cannot be modified through API operations.

Field security enables selective data exposure through APIs without requiring separate table structures. Sensitive fields can be protected while allowing access to general business data.

Consider field permission inheritance and dependencies when designing API security models. Related fields or calculated fields may require additional permission considerations.

## Operation-Specific Permissions

Different OData operations require specific permission configurations. GET operations require read permissions, POST operations require insert permissions, and PATCH/PUT operations require modify permissions.

DELETE operations through API Pages require explicit delete permissions on the source table. Not all API implementations need to support delete operations, allowing for read-only or append-only API designs.

Permission requirements may vary based on API operation complexity. Operations that modify multiple tables or trigger business logic may require additional permission grants.

## Service Account Security

API integrations typically use dedicated service accounts with restricted permission sets designed for specific integration scenarios. Service accounts should follow security best practices for automated system access.

Service account permissions should be regularly reviewed and maintained to ensure they remain appropriate for current integration requirements. Unused permissions should be removed to minimize security exposure.

Consider using Azure AD application registrations for API authentication when possible. This provides better audit trails and credential management compared to traditional BC user accounts.

## Authentication and Authorization Flow

API Page security integrates with BC authentication mechanisms including basic authentication, OAuth 2.0, and Azure AD authentication. Choose authentication methods appropriate for security requirements and integration scenarios.

Authorization occurs after successful authentication, verifying that the authenticated account has appropriate permissions for the requested API operations. Failed authorization results in access denied responses.

Implement proper error handling for authentication and authorization failures to provide appropriate feedback to API consumers while avoiding security information disclosure.

## Multi-Tenancy Security Considerations

In multi-tenant environments, permission models must ensure proper tenant isolation. API consumers should only access data within their authorized tenant boundaries.

Permission sets may need tenant-specific configurations to address different security requirements across tenants. Consider centralized permission management approaches for consistency.

Monitor cross-tenant access patterns to detect potential security issues or configuration errors that could compromise tenant data isolation.

## Compliance and Audit Requirements

API permission models must support compliance requirements including data access auditing, permission change tracking, and access review processes.

Implement proper logging for API access patterns to support security monitoring and compliance reporting. Log both successful and failed access attempts for comprehensive audit trails.

Consider implementing automated permission review processes to ensure API access remains appropriate over time and changes in business requirements.

## Dynamic Permission Scenarios

Some API implementations require dynamic permission evaluation based on data values, business logic, or contextual factors beyond standard BC permission sets.

Dynamic permissions may be implemented through custom business logic in API Pages or source tables. This enables complex security scenarios but requires careful implementation to avoid security vulnerabilities.

Consider performance impact of dynamic permission evaluation, especially for high-volume API scenarios where permission checks could become performance bottlenecks.

## Security Monitoring and Alerting

Implement monitoring for unusual API access patterns that might indicate security issues or unauthorized access attempts. Monitor both successful and failed API requests for security indicators.

Set up alerting for permission-related errors or unusual access patterns that require investigation. Early detection of security issues enables rapid response and mitigation.

Regular security reviews should include analysis of API access patterns and permission effectiveness to identify optimization opportunities and potential security improvements.

## Integration with External Security Systems

API permission models may need to integrate with external security systems, identity providers, or access management solutions. Plan integration approaches that maintain security effectiveness.

Consider federated identity scenarios where API consumers authenticate through external systems but require BC permissions for data access. Implement proper token validation and permission mapping.

External security integration should maintain audit trails and support compliance requirements while providing seamless authentication experiences for API consumers.

## Best Practices

Follow the principle of least privilege when designing API permission models. Grant only the minimum permissions required for legitimate API operations.

Implement defense in depth through multiple security layers including authentication, authorization, field-level security, and business logic validation.

Document permission requirements and security assumptions for API implementations. This documentation supports security reviews and helps maintain consistent security practices.

Regularly review and validate API permission configurations to ensure they remain appropriate for current business requirements and security standards.