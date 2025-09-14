---
title: "Business Central API Error Response Patterns"
domain: "api-design"
difficulty: "intermediate"
bc_versions: "14+"
tags: ["error-handling", "http-status", "api-responses"]
samples: "samples/api-error-responses.md"
---

# Business Central API Error Response Patterns

## Overview

Business Central API error responses follow OData standards while providing BC-specific error context. Understanding these patterns is crucial for building robust integrations that can gracefully handle various error scenarios.

Proper error response design helps API consumers implement appropriate retry logic, user messaging, and debugging capabilities, directly impacting integration reliability and maintainability.

## Standard Error Response Structure

### OData Error Format
BC APIs return errors in standard OData format with error code, message, and additional context. This consistency enables generic error handling patterns across different BC APIs.

### Error Code Categories
BC uses specific error code patterns to categorize different error types, allowing consuming applications to implement targeted handling strategies based on error classification.

### Message Localization
Error messages are returned in the user's language context when possible, though integration scenarios typically require language-neutral error handling approaches.

## HTTP Status Code Patterns

### 400 Bad Request Usage
Returned for malformed requests, invalid field values, or business rule violations. Indicates client-side issues that require request modification before retry.

### 401 Unauthorized Scenarios
Authentication failures and expired tokens trigger 401 responses. Consuming applications should implement token refresh logic for sustainable integration.

### 403 Forbidden Applications
Permission-related errors where authentication succeeds but access is denied. Requires administrative intervention rather than programmatic resolution.

### 404 Not Found Contexts
Resource not found scenarios including deleted records, invalid entity references, or incorrect URL patterns requiring client-side validation enhancement.

### 409 Conflict Situations
Concurrency conflicts and business rule violations that prevent operation completion. Often includes ETag-related conflicts requiring client retry with updated data.

### 500 Internal Server Error
System-level errors indicating BC platform issues. Consuming applications should implement exponential backoff retry patterns for transient failures.

## Business Rule Error Handling

### Validation Error Responses
Field validation failures include specific field references and violation descriptions, enabling targeted user interface feedback in consuming applications.

### Cross-Field Validation Errors
Business rules spanning multiple fields return comprehensive error descriptions with context about all affected fields and their relationships.

### Custom Validation Integration
Custom AL validation code integrates seamlessly with standard error response patterns, maintaining consistency across built-in and custom business rules.

## Concurrency and ETag Errors

### Optimistic Concurrency Failures
ETag mismatches result in 409 Conflict responses with details about the current record state, enabling intelligent conflict resolution in consuming applications.

### Concurrent Modification Detection
BC detects concurrent modifications and provides detailed information about conflicting changes to support merge conflict resolution strategies.

### Recovery Pattern Guidance
Error responses include sufficient context for implementing automatic recovery patterns like refresh-and-retry for simple concurrency conflicts.

## Permission and Security Errors

### Insufficient Permission Context
Permission errors specify the required permission level and affected resource, helping administrators diagnose and resolve access issues efficiently.

### License Limitation Errors
BC license limitations generate specific error patterns that consuming applications can use to provide appropriate user guidance about feature availability.

### Security Policy Violations
Data access policy violations include context about the specific security constraint that prevented operation completion.

## Integration-Specific Error Patterns

### Webhook Delivery Failures
Webhook scenarios generate specific error patterns for subscription management, delivery failures, and endpoint validation issues.

### Batch Operation Errors
Multi-record operations return detailed error information for each failed item while maintaining overall operation context and partial success indicators.

### Cross-Company Access Errors
Multi-company API scenarios generate specific error patterns for company selection, cross-company permission, and data isolation violations.

## Error Recovery Strategies

### Transient Error Identification
BC error responses include indicators for transient vs permanent failures, enabling appropriate retry logic implementation in consuming applications.

### Retry-After Guidance
Rate limiting and throttling scenarios include Retry-After headers with specific timing guidance for sustainable integration patterns.

### Diagnostic Information
Error responses include sufficient diagnostic context for troubleshooting without exposing sensitive system information or security details.

## Best Practices for Error Handling

### Client-Side Error Processing
Implement structured error processing that leverages BC error code patterns for automated categorization and appropriate response strategies.

### Logging and Monitoring
Capture error patterns for monitoring integration health and identifying systemic issues requiring attention from BC administrators.

### User Experience Considerations
Transform technical error responses into user-friendly messages while preserving technical details for support and debugging scenarios.

### Graceful Degradation
Design integration flows to gracefully handle various error scenarios without causing complete system failures or data inconsistency issues.