---
title: "ETag Implementation for Optimistic Concurrency"
domain: "concurrency"
difficulty: "advanced"
bc_versions: "14+"
tags: ["etag", "concurrency", "conflict-resolution"]
samples: "samples/etag-implementation.md"
---

# ETag Implementation for Optimistic Concurrency

## Overview

ETag implementation in Business Central API Pages provides optimistic concurrency control to prevent data conflicts in multi-user scenarios. Understanding ETag patterns enables robust API implementations that handle concurrent access gracefully while maintaining data integrity.

ETags are HTTP headers that represent the current version of a resource, allowing clients to perform conditional operations based on resource state. BC automatically generates ETags for API Pages based on record modification information.

Proper ETag implementation prevents lost updates, enables conflict detection, and supports distributed API scenarios where multiple clients may simultaneously access the same data resources.

## ETag Generation Mechanisms

Business Central automatically generates ETag values for API Page records based on internal record versioning information. These values change whenever the underlying record is modified through any mechanism.

ETag values are typically based on timestamp information, system version numbers, or hash calculations that uniquely represent the current state of the record. The specific implementation depends on table configuration and BC version.

ETag generation is transparent to API implementations but understanding the underlying mechanisms helps design effective concurrency control strategies for specific business scenarios.

## HTTP Conditional Operations

ETag values enable HTTP conditional operations through If-Match and If-None-Match headers. These headers allow clients to specify conditions for request processing based on resource state.

If-Match headers ensure operations only proceed if the resource hasn't changed since the client last retrieved it. This prevents overwriting changes made by other clients during the same time period.

If-None-Match headers enable efficient caching scenarios where clients only retrieve resources that have changed since their last request, reducing bandwidth and processing overhead.

## Client-Side Implementation Patterns

API clients should retrieve and store ETag values when reading resources through API Pages. These values must be included in subsequent modification requests to enable proper concurrency control.

Implement proper error handling for ETag conflicts, including retry mechanisms and user notification strategies that provide appropriate feedback when concurrent modification conflicts occur.

Consider client-side caching strategies that leverage ETag values to minimize unnecessary API requests and improve overall application performance.

## Conflict Detection and Resolution

ETag mismatches indicate concurrent modification conflicts that require resolution before proceeding with requested operations. BC returns appropriate HTTP status codes to indicate conflict situations.

Implement proper conflict resolution strategies including automatic retry with fresh data, user-prompted conflict resolution, or last-writer-wins approaches based on business requirements.

Consider the user experience implications of conflict resolution, providing clear feedback and options when conflicts occur in interactive API scenarios.

## Performance Implications

ETag validation adds minimal overhead to API operations while providing significant benefits for data integrity and conflict prevention. The performance impact is typically negligible compared to conflict resolution costs.

Proper ETag implementation can improve overall system performance by preventing unnecessary data processing when resources haven't changed since last access.

Monitor ETag-related performance metrics to ensure concurrency control mechanisms don't negatively impact overall API performance under high-load scenarios.

## Multi-User Scenario Handling

ETag implementation becomes critical in multi-user scenarios where multiple clients may simultaneously modify the same data resources. Without proper concurrency control, data corruption or lost updates can occur.

Design API workflows that account for concurrent access patterns, implementing appropriate retry mechanisms and user feedback for conflict situations.

Consider the business impact of concurrent modification scenarios and design appropriate resolution strategies that align with business requirements and user expectations.

## Integration with Business Logic

ETag validation occurs before business logic execution, preventing unnecessary processing when concurrent modification conflicts exist. This provides efficient conflict detection with minimal resource consumption.

Business logic implementations should consider ETag behavior and design operations that work effectively with optimistic concurrency control patterns.

Complex business operations involving multiple records may require advanced concurrency strategies beyond simple ETag validation to ensure complete transaction consistency.

## Error Handling Strategies

Implement comprehensive error handling for ETag-related scenarios including missing ETags, invalid ETag formats, and concurrency conflict situations.

Provide meaningful error messages that help API clients understand and respond appropriately to concurrency-related issues without exposing sensitive system information.

Consider automated retry strategies for transient concurrency conflicts while avoiding infinite retry loops that could impact system performance.

## Testing Concurrency Scenarios

Implement thorough testing for concurrent access scenarios including simultaneous modification attempts by multiple clients. These scenarios can be difficult to reproduce consistently but are critical for robust API implementations.

Use automated testing tools to simulate concurrent access patterns and validate ETag behavior under realistic load conditions and timing scenarios.

Test edge cases including rapid successive modifications, network delays affecting ETag delivery, and client-side caching interactions with ETag validation.

## Advanced ETag Patterns

Consider weak versus strong ETag implementations for different business scenarios. Weak ETags provide flexibility for semantically equivalent representations while strong ETags ensure byte-level consistency.

Implement custom ETag generation strategies for complex business scenarios that require specific versioning behavior or integration with external versioning systems.

Advanced scenarios may require ETag integration with distributed caching systems or complex business workflow patterns that span multiple API operations.

## Security Considerations

ETag values should not expose sensitive information about internal system state or business data. Ensure ETag generation mechanisms maintain appropriate security boundaries.

Consider the security implications of ETag-based caching scenarios, ensuring cached resources don't bypass appropriate security controls or expose unauthorized data.

Monitor for potential security issues related to ETag manipulation or exploitation that could affect system security or data integrity.

## Best Practices

Always implement ETag support for API Pages that support modification operations. The concurrency control benefits significantly outweigh the minimal implementation overhead.

Design client applications to properly handle ETag workflows including storage, transmission, and conflict resolution scenarios for robust API integration.

Document ETag behavior and requirements for API consumers, providing clear guidance on proper implementation patterns and conflict resolution strategies.

Monitor and analyze ETag conflict patterns to identify potential optimization opportunities in business processes or API design that could reduce concurrent modification scenarios.