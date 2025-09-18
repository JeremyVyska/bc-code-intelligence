---
title: "Custom Telemetry Implementation in AL"
domain: "dean-debug"
difficulty: "intermediate"
bc_versions: "14+"
tags: ["telemetry", "session-logmessage", "privacy", "diagnostics", "monitoring"]
samples: "samples/custom-telemetry.md"
type: "implementation-pattern"
category: "diagnostics"
pattern_type: "good"
severity: "medium"
impact_level: "high"
improvement_suggestion: "Implement custom telemetry for actionable monitoring and diagnostics"
implementation_steps: ["design-telemetry-schema", "implement-session-logmessage", "validate-privacy-compliance"]
validation_criteria: ["privacy-compliant", "actionable-data", "performance-neutral"]
technical_areas: ["al-code", "monitoring"]
---

# Custom Telemetry Implementation in AL

## Overview

Custom telemetry in AL enables developers to emit structured diagnostic data through Session.LogMessage() for Application Insights integration. This observability foundation supports proactive monitoring, performance analysis, and troubleshooting while maintaining strict privacy compliance.

**Critical Principle**: Treat telemetry as an API - documented, versioned, discoverable, and non-breaking for downstream analytics consumers.

## Strategic Framework

### Telemetry Design Philosophy
- **Actionable Data Only**: Emit signals customers can act upon, avoiding noise from non-actionable metrics
- **Privacy by Design**: Never log PII, customer data, or sensitive business information
- **Cost Consciousness**: Balance diagnostic value with data ingestion costs through intelligent sampling
- **Schema Consistency**: Use standardized naming conventions and structured dimensions

### When to Implement Custom Telemetry
- **Error Deflection**: Authentication failures, permission issues, configuration problems
- **Performance Monitoring**: Long-running operations, resource bottlenecks, timeout scenarios
- **Feature Adoption**: Usage patterns, workflow completion rates, feature discovery
- **Integration Health**: External service calls, API responses, data synchronization status

## Architecture Focus

### Session.LogMessage() Implementation Pattern
Structure telemetry calls with consistent event IDs, descriptive messages following "Object ActionInPastTense" pattern, and appropriate verbosity levels. Use custom dimensions dictionary for structured data while maintaining PascalCasing conventions.

### Privacy-Compliant Data Classification
Always use DataClassification::SystemMetadata for telemetry events. Review all LogMessage calls for potential privacy violations before production deployment. Focus on operational metadata rather than business data.

### Telemetry Scope Strategy
Default to TelemetryScope::ExtensionPublisher to control costs. Use TelemetryScope::All only when customers can directly act on the telemetry data for their business operations.

## Best Practices

### Implementation Guidelines
- **Object-Level Emission**: Call Session.LogMessage() from within objects causing situations requiring telemetry
- **Structured Dimensions**: Use Dictionary of [Text,Text] for custom dimensions with meaningful prefixes
- **Message Consistency**: Always include human-readable "message" field expressing event purpose
- **Correlation Support**: Include identifiers enabling distributed tracing across system boundaries

### Performance Considerations
- **Non-Blocking**: Ensure telemetry calls never impact user experience or application performance
- **Minimal Processing**: Avoid complex calculations within telemetry emission paths
- **Sampling Strategy**: Implement intelligent sampling for high-frequency events

### Analytics Integration
Design telemetry for Application Insights consumption with KQL-friendly field names, consistent schemas across related events, and dimensions supporting common analytical queries.

## Anti-Patterns

### Avoid These Approaches
- **Privacy Violations**: Logging customer data, PII, or sensitive business information
- **Telemetry Flooding**: Excessive volume without considering customer ingestion costs
- **Functional Dependencies**: Using telemetry data for application logic rather than pure observability
- **Inconsistent Schemas**: Different naming patterns across related telemetry events

### Common Implementation Mistakes
- Missing privacy review before production deployment
- Using special characters or spaces in custom dimension keys
- Emitting telemetry from central objects rather than originating sources
- Failing to provide actionable information for intended telemetry consumers

*Implementation examples: see samples/custom-telemetry.md*
*Privacy guidelines: see privacy-compliance-patterns.md*
*Related patterns: monitoring-strategy.md, error-handling-patterns.md*