---
title: "BC Session LogMessage Methods"
domain: "dean-debug"
difficulty: "intermediate"
bc_versions: "14+"
tags: ["telemetry", "logging", "session", "performance"]
prerequisites: ["telemetry-basics"]
samples: "samples/logmessage-methods.md"
---
# BC Session LogMessage Methods

## Overview

Business Central's Session.LogMessage() provides structured telemetry logging with multiple overloads for different verbosity levels and correlation scenarios. Understanding the method variations and their performance implications is essential for effective telemetry implementation.

The LogMessage methods enable developers to emit custom telemetry events that integrate with Application Insights and other monitoring solutions. Each method variation serves specific use cases from simple informational logging to complex error correlation.

## Method Variations

### Basic LogMessage
The fundamental LogMessage(Message, Verbosity) method provides simple event logging with verbosity control. This method creates telemetry events with basic message content and system-generated correlation identifiers.

Verbosity levels range from Critical (1) through Error (2), Warning (3), Normal (4), and Verbose (5). Higher verbosity numbers indicate less critical information and may be filtered in production environments.

### LogMessage with Custom Dimensions
The LogMessage(Message, Verbosity, DataClassification, TelemetryScope, Dictionary) overload enables rich telemetry with custom dimensions. This method allows developers to include structured data that enhances telemetry analysis and correlation.

Custom dimensions support key-value pairs that appear as structured fields in Application Insights. This enables sophisticated querying, filtering, and correlation analysis in telemetry dashboards.

### LogMessage with Correlation
Advanced LogMessage overloads include correlation parameters that link related telemetry events across different system components. This correlation capability is essential for tracing complex business processes and debugging distributed operations.

## Performance Characteristics

### Memory Impact
LogMessage operations consume memory for message formatting and custom dimension serialization. Large custom dimension collections or frequent high-verbosity logging can impact system memory utilization.

The telemetry system buffers events before transmission, so sustained high-volume logging may increase memory pressure. Consider telemetry volume in memory-constrained environments.

### CPU Overhead
Message formatting and custom dimension processing consume CPU resources. Complex custom dimensions or frequent logging calls contribute to overall system CPU utilization.

Verbosity-based filtering occurs after message formatting, so high-verbosity messages still incur formatting costs even when filtered. Design telemetry strategies to minimize unnecessary processing.

### Network and Storage
Telemetry events consume network bandwidth for transmission to monitoring systems and storage space in telemetry databases. High-volume telemetry can impact both network performance and storage costs.

Consider telemetry retention policies and sampling strategies to balance diagnostic capability with resource consumption.

## Best Practices

### Verbosity Selection
Choose appropriate verbosity levels based on information criticality and expected production filtering. Use Critical/Error for system issues, Warning for recoverable problems, Normal for business events, and Verbose for detailed diagnostics.

### Custom Dimension Design
Structure custom dimensions for effective querying and analysis. Use consistent naming conventions and avoid high-cardinality values that complicate aggregation analysis.

### Correlation Strategy
Implement consistent correlation patterns across related components to enable effective trace analysis. Use business process identifiers where possible to link telemetry events to actual user scenarios.