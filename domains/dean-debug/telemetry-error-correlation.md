---
title: "Telemetry Error Correlation"
domain: "dean-debug"
difficulty: "intermediate"
bc_versions: "14+"
tags: ["telemetry", "debugging", "correlation", "troubleshooting"]
prerequisites: ["bc-session-logmessage-methods"]
samples: "samples/error-correlation.md"
---
# Telemetry Error Correlation

## Overview

Error correlation links related telemetry events across different system components to provide complete context for troubleshooting. Effective correlation enables rapid root cause analysis by connecting error symptoms with underlying causes across complex business processes.

Business Central's telemetry correlation mechanisms support both automatic session-based correlation and custom correlation identifiers that span multiple sessions or system boundaries.

## Correlation Mechanisms

### Session-Based Correlation
Business Central automatically correlates telemetry events within the same user session using system-generated session identifiers. This automatic correlation links events from the same user interaction across different codeunits and operations.

Session correlation works effectively for single-user operations but has limitations for background processes, integration scenarios, or operations that span multiple sessions.

### Custom Correlation Identifiers
Custom correlation identifiers enable linking related events across session boundaries. These identifiers can represent business processes, document workflows, or integration operations that span multiple system interactions.

Implement custom correlation using consistent identifier generation and inclusion in custom dimensions across all related telemetry events. Choose correlation identifiers that meaningfully represent business processes or technical operations.

### Operation Context Correlation
Operation-level correlation links events within specific business operations like document posting, report generation, or integration processing. This correlation level bridges the gap between session correlation and business process correlation.

## Implementation Patterns

### Error Event Design
Structure error telemetry events to include comprehensive context for troubleshooting. Include business context (document numbers, customer identifiers), technical context (component names, operation types), and correlation context (session, operation, process identifiers).

Design error events to capture both immediate error conditions and related context that supports root cause analysis. Consider including preceding events or system state information that helps explain error conditions.

### Correlation Propagation
Propagate correlation identifiers consistently across all components involved in business processes. Ensure correlation identifiers flow through integration boundaries, background job execution, and external system interactions.

Use parameter passing, global variables, or context objects to maintain correlation identifiers throughout business process execution. Design correlation propagation to be resilient to error conditions and system boundaries.

### Multi-Level Correlation
Implement multiple correlation levels to support different troubleshooting scenarios. Combine session-level, operation-level, and business process-level correlation to enable both detailed technical analysis and high-level business impact assessment.

## Troubleshooting Applications

### Root Cause Analysis
Use correlation identifiers to trace error conditions back to their originating causes. Link error symptoms visible to users with underlying technical issues across multiple system components.

Correlation enables analysis of error cascades where initial problems trigger subsequent failures across related components or business processes.

### Integration Debugging
Correlation is essential for debugging integration scenarios where errors may occur in external systems but manifest as Business Central issues. Link Business Central telemetry with external system logs using shared correlation identifiers.

### Performance Issue Investigation
Use correlation to link performance symptoms with underlying causes. Correlate user-reported performance issues with system resource utilization, database performance, or integration latency across multiple components.

## Advanced Correlation Patterns

### Hierarchical Correlation
Implement hierarchical correlation structures that represent complex business processes with multiple sub-operations. Use parent-child correlation relationships to organize telemetry events into logical groupings.

### Cross-System Correlation
Extend correlation across system boundaries to link Business Central events with external system activities. Include shared correlation identifiers in integration messages and API calls to enable comprehensive troubleshooting.

### Temporal Correlation
Consider time-based correlation patterns that link events occurring within specific time windows. This approach helps identify related issues that may not share explicit correlation identifiers.

## Analysis and Tooling

### Correlation Queries
Design Application Insights queries that leverage correlation identifiers for effective troubleshooting analysis. Create saved queries and dashboards that utilize correlation patterns for routine troubleshooting tasks.

### Automated Correlation Analysis
Consider implementing automated analysis that identifies correlation patterns and potential root causes. Use correlation data to build troubleshooting decision trees and diagnostic workflows.