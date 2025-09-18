---
title: "Custom Dimensions Best Practices"
domain: "dean-debug"
difficulty: "intermediate"
bc_versions: "14+"
tags: ["telemetry", "dimensions", "analytics", "monitoring"]
prerequisites: ["bc-session-logmessage-methods"]
samples: "samples/custom-dimensions.md"
---
# Custom Dimensions Best Practices

## Overview

Custom dimensions transform basic telemetry events into rich, queryable data that enables sophisticated analysis and monitoring. Proper custom dimension design balances diagnostic capability with performance impact and storage efficiency.

Custom dimensions appear as structured fields in Application Insights and other monitoring solutions, enabling filtering, grouping, and correlation analysis. Well-designed dimensions enhance troubleshooting capabilities and business intelligence extraction from telemetry data.

## Dimension Design Principles

### Consistency and Standardization
Establish consistent naming conventions across all telemetry events. Use standardized dimension names for common concepts like user identification, operation context, and business entity references.

Consistent dimension naming enables effective cross-component analysis and simplifies dashboard creation. Consider creating a telemetry dimension registry to maintain naming standards across development teams.

### Cardinality Management
Avoid high-cardinality dimensions that create excessive unique value combinations. High cardinality impacts query performance, storage efficiency, and aggregation capabilities in monitoring systems.

Examples of high-cardinality dimensions include timestamps, GUIDs, or detailed error messages. Instead, categorize high-cardinality values or use separate correlation mechanisms for detailed tracking.

### Business Context Integration
Include dimensions that align with business processes and operational concerns. Business-relevant dimensions enable telemetry analysis that directly supports troubleshooting and process optimization.

Consider dimensions for customer identification, transaction types, document numbers, and business process stages. These dimensions link technical telemetry to business impact analysis.

## Implementation Strategies

### Dimension Categories
Organize custom dimensions into logical categories: technical context (component, operation), business context (customer, document), and correlation context (session, trace identifiers).

Technical dimensions support system troubleshooting, business dimensions enable process analysis, and correlation dimensions link related events across system boundaries.

### Performance Optimization
Minimize custom dimension processing overhead by pre-formatting dimension values and reusing dimension collections where appropriate. Avoid complex calculations or expensive operations within dimension value generation.

Consider dimension value caching for frequently-used values and lazy evaluation for expensive dimension calculations that may not be needed in all scenarios.

### Data Classification Alignment
Ensure custom dimension content aligns with Business Central data classification requirements. Avoid including sensitive data in telemetry dimensions without appropriate classification and protection measures.

## Common Anti-Patterns

### Excessive Dimensions
Including too many custom dimensions per telemetry event creates storage bloat and complicates analysis. Focus on dimensions that provide actual analytical value rather than comprehensive data capture.

### Dynamic Dimension Names
Creating dimension names dynamically based on runtime values prevents effective querying and analysis. Use consistent dimension names with variable values rather than variable dimension names.

### Sensitive Data Inclusion
Including customer data, financial information, or other sensitive content in custom dimensions violates data protection principles and may create compliance issues.

## Analysis and Querying

### Dashboard Design
Structure custom dimensions to support effective dashboard creation and alerting rules. Consider how dimensions will be used in aggregation queries and filtering operations.

### Correlation Patterns
Design custom dimensions to enable effective event correlation across different system components. Include consistent correlation identifiers that link related business processes.

### Time-Series Analysis
Include dimensions that support time-series analysis of business processes and system performance. Consider dimensions for business cycles, operational periods, and performance trends.