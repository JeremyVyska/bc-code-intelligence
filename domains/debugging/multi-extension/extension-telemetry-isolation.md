---
title: "Extension Telemetry Isolation"
domain: "debugging"
difficulty: "intermediate"
bc_versions: "14+"
tags: ["telemetry", "extensions", "isolation", "multi-tenancy"]
prerequisites: ["custom-dimensions-best-practices"]
samples: "samples/extension-isolation.md"
---

# Extension Telemetry Isolation

## Overview

Extension telemetry isolation ensures that telemetry events from different extensions can be independently monitored, analyzed, and troubleshot without interference. Proper isolation enables extension publishers to monitor their components separately while supporting comprehensive system-wide analysis.

Telemetry isolation becomes critical in multi-extension environments where different publishers need independent monitoring capabilities and where system administrators require the ability to analyze issues by extension scope.

## Isolation Strategies

### Extension Identification
Include consistent extension identification in all telemetry events through custom dimensions. Use extension publisher information, extension name, and version details to enable filtering and analysis by extension scope.

Implement standardized extension identification patterns that work across all extensions and enable consistent filtering in monitoring systems. Consider using the extension's App ID as a reliable identification mechanism.

### Namespace Separation
Design telemetry event naming conventions that include extension context to prevent naming conflicts between extensions. Use prefixed event names or structured naming that clearly identifies the source extension.

Namespace separation enables independent analysis while preventing confusion when multiple extensions emit similar telemetry event types.

### Custom Dimension Isolation
Use extension-specific custom dimension naming to prevent dimension name conflicts between extensions. Implement dimension naming conventions that include extension context while maintaining analytical capability.

Consider using dimension namespaces or prefixes that clearly identify the source extension while enabling cross-extension analysis where appropriate.

## Multi-Tenant Considerations

### Tenant Isolation
Ensure telemetry isolation works effectively in multi-tenant environments where multiple tenants may use the same extensions. Combine extension isolation with tenant identification to enable both extension-specific and tenant-specific analysis.

Design isolation patterns that scale across large multi-tenant deployments while maintaining performance and analytical capability.

### Cross-Tenant Analysis
Consider scenarios where extension publishers need to analyze telemetry across multiple tenant installations. Design isolation patterns that enable this analysis while respecting tenant data privacy and security requirements.

## Implementation Patterns

### Extension Context Propagation
Propagate extension identification consistently throughout business process execution, including scenarios where multiple extensions interact. Ensure extension context flows through integration boundaries and system interactions.

Use parameter passing or context objects to maintain extension identification throughout complex business processes that span multiple components or extensions.

### Shared Resource Monitoring
Design telemetry patterns for monitoring shared resources (like database tables or integration endpoints) that are accessed by multiple extensions. Enable analysis by extension while maintaining visibility into overall resource utilization.

### Extension Interaction Tracking
Implement telemetry patterns that track interactions between different extensions. This capability supports troubleshooting scenarios where issues arise from extension interactions or dependencies.

## Analysis and Monitoring

### Extension-Specific Dashboards
Create monitoring dashboards that filter telemetry by extension scope. Enable extension publishers to monitor their components independently while providing system administrators with comprehensive system views.

### Cross-Extension Analysis
Design analysis capabilities that can examine interactions between extensions while respecting isolation boundaries. Support troubleshooting scenarios where issues span multiple extensions.

### Performance Impact Isolation
Enable analysis of performance impact by extension to identify which extensions contribute to system performance issues. Use extension identification to isolate performance telemetry and enable targeted optimization efforts.

## Troubleshooting Applications

### Extension-Specific Issues
Use extension isolation to focus troubleshooting efforts on specific extensions when issues are identified. Filter telemetry to specific extensions to reduce noise and accelerate root cause identification.

### Integration Issue Analysis
Leverage extension isolation when troubleshooting integration issues that may involve multiple extensions. Analyze telemetry from each involved extension to understand interaction patterns and identify failure points.

### Performance Investigation
Use extension isolation to identify performance bottlenecks attributable to specific extensions. Analyze resource utilization and operation timing by extension scope to guide optimization efforts.