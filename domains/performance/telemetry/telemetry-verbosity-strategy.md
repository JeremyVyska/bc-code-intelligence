---
title: "Telemetry Verbosity Strategy"
domain: performance
type: knowledge-topic
category: performance
severity: low
name: telemetry-verbosity-strategy
description: "Strategy for managing telemetry verbosity levels in production environments"
impact_level: low
detection_confidence: medium
difficulty: beginner
bc_versions: "14+"
tags: ["telemetry", "verbosity", "production", "diagnostics"]
prerequisites: ["bc-session-logmessage-methods"]
samples: "samples/verbosity-strategy.md"
---

# Telemetry Verbosity Strategy

## Overview

Telemetry verbosity controls determine which events are captured and transmitted to monitoring systems. Effective verbosity strategy balances diagnostic capability with system performance and storage costs.

Business Central provides five verbosity levels from Critical (1) to Verbose (5). Production environments typically filter higher verbosity levels to reduce telemetry volume while preserving essential diagnostic information.

## Verbosity Levels

### Critical (Level 1)
Critical events indicate system failures or conditions that prevent normal operation. These events should always be captured and typically trigger immediate alerting in production environments.

Use Critical verbosity for unrecoverable errors, system component failures, and conditions that impact business continuity. Critical events warrant immediate attention and investigation.

### Error (Level 2)
Error events indicate recoverable problems that impact specific operations but don't prevent overall system function. These events should be captured in production for troubleshooting and trend analysis.

Use Error verbosity for operation failures, data validation issues, and integration problems that affect individual transactions or users.

### Warning (Level 3)
Warning events indicate potential problems or unusual conditions that don't immediately impact operations. These events help identify emerging issues before they become critical problems.

Use Warning verbosity for performance degradation, configuration issues, and deprecated functionality usage that may require attention.

### Normal (Level 4)
Normal events capture significant business operations and system activities for operational monitoring and business intelligence. These events provide insight into system usage and business process execution.

Use Normal verbosity for successful business transactions, user actions, and integration activities that support operational analysis.

### Verbose (Level 5)
Verbose events provide detailed diagnostic information for troubleshooting and development scenarios. These events are typically filtered in production environments due to volume considerations.

Use Verbose verbosity for detailed execution traces, debugging information, and granular operational details needed for development and testing scenarios.

## Production Strategy

### Baseline Configuration
Establish production verbosity baselines that capture essential diagnostic information without overwhelming monitoring systems. Typical production configurations capture Critical through Normal levels.

Consider business requirements, troubleshooting needs, and storage capacity when establishing baseline verbosity levels. Adjust baselines based on operational experience and incident analysis needs.

### Dynamic Adjustment
Implement capabilities to adjust verbosity levels dynamically during incident investigation or troubleshooting scenarios. Temporary verbosity increases can provide additional diagnostic information without permanent overhead.

Consider implementing verbosity level controls that can be adjusted per component or business process to focus additional telemetry on specific problem areas.

## Development Strategy

### Testing and Development
Use higher verbosity levels during development and testing to capture detailed execution information. Verbose telemetry supports debugging, performance analysis, and integration testing scenarios.

Consider separate verbosity configurations for different environments to optimize telemetry capture for each environment's specific needs.

### Performance Testing
Include telemetry verbosity impact in performance testing scenarios. High verbosity levels can significantly impact system performance and should be considered in capacity planning.

## Implementation Considerations

### Performance Impact
Higher verbosity levels increase CPU, memory, and network utilization for telemetry processing and transmission. Consider telemetry overhead in overall system performance planning.

### Storage and Cost Management
Telemetry volume directly impacts monitoring system storage requirements and costs. Balance diagnostic capability with storage capacity and cost constraints.

### Filtering and Sampling
Consider implementing telemetry sampling strategies for high-volume scenarios to reduce data volume while preserving statistical significance for analysis.