---
title: "Case Statement Multiple Conditions and Ranges"
domain: "language-fundamentals"
subdomain: "control-structures"
difficulty: "intermediate"
bc_versions: "14+"
al_version: "6.0+"
tags: ["case", "ranges", "multiple-values", "advanced-patterns", "conditional-logic"]
prerequisites: ["case-statement-syntax", "data-types", "operators"]
related_topics: ["case-statement-syntax", "case-performance", "numeric-operations"]
samples: "samples/case-statement-multiple-conditions.md"
---

# Case Statement Multiple Conditions and Ranges

## Overview

AL Case statements support sophisticated pattern matching beyond simple value comparison. Multiple values and numeric ranges enable elegant solutions for classification, tiering, and categorical logic common in Business Central applications.

**Key Capabilities:**
- **Multiple Values**: Single branch handling multiple discrete values
- **Numeric Ranges**: Continuous value ranges using dotdot (..) operator
- **Mixed Patterns**: Combining discrete values and ranges in same statement
- **Type-Safe Matching**: Compile-time validation of value compatibility

These advanced patterns eliminate complex IF-THEN chains and provide more maintainable conditional logic.

## Multiple Value Syntax

### Comma-Separated Values
Single case branches can handle multiple discrete values using comma-separated syntax. This pattern groups values that require identical processing logic.

### Enum Multiple Values
Enum values can be grouped using comma separation for branches that process multiple related document types or states with common handling logic.

### Best Practices for Multiple Values
- **Logical Grouping**: Group values that require identical processing
- **Clear Relationships**: Ensure grouped values share common business logic
- **Maintainability**: Document why specific values are grouped together

## Numeric Range Operations

### Basic Range Syntax
The dotdot (..) operator creates inclusive ranges for numeric types. Range bounds include both start and end values, covering all integers between specified limits.

### Range Characteristics
- **Inclusive Bounds**: Both start and end values are included in the range
- **Continuous Coverage**: Ranges cover all integer values between bounds
- **Type Requirements**: Range bounds must be compatible numeric types
- **Compile-Time Validation**: Invalid ranges detected during compilation

## Advanced Range Patterns

### Age-Based Classification
Age ranges enable automatic classification into demographic categories with appropriate processing for minors, adults, and seniors.

### Price Tier Determination  
Amount ranges determine discount percentages based on order values, with higher volumes receiving increased discount rates.

### Scoring and Grading
Test score ranges automatically assign letter grades using standard academic grading scales with error handling for invalid scores.

## Mixed Pattern Implementation

### Combining Discrete Values and Ranges
Single case statements can mix discrete string values with numeric ranges, enabling flexible processing based on different data types within the same logic structure.

### Complex Business Logic
Complex business scenarios combine multiple value types with multi-statement branches to handle VIP customers, loyalty levels, and default processing paths.

## Range Validation and Error Handling

### Gap Detection
Ensure complete coverage of expected value ranges by designing consecutive ranges that cover all valid input values without gaps.

### Overlap Prevention
Avoid overlapping ranges that create ambiguous conditions where multiple branches could match the same input value.

## Performance Considerations

### Range Evaluation Efficiency
- **Ordered Ranges**: Place most common ranges first for optimal performance
- **Large Ranges**: Consider breaking very large ranges into smaller segments
- **Range Density**: Dense ranges (consecutive values) perform better than sparse ranges

### Optimization Strategies
Order case branches by frequency, placing most common ranges first to minimize average execution time through reduced branch traversal.

## AL-Specific Implementation Notes

### Data Type Compatibility
- **Integer Ranges**: Support full integer range with MAXINTEGER constant
- **Decimal Ranges**: Limited precision considerations for decimal ranges
- **Date Ranges**: Supported but consider DateTime for time-sensitive logic
- **Time Ranges**: Available for time-based processing windows

### Constants and Variables
Use constants for maintainable range bounds to centralize business rules and enable easy modification of threshold values across the application.

## CodeCop Compliance Considerations

### Official AL Guidelines Conflict

**Important Note**: The patterns described in this topic conflict with Microsoft's CodeCop rule AA0005, which governs the use of BEGIN..END blocks in case statements.

This represents a common styling debate in AL development:

#### AA0005: BEGIN..END Usage in Case Branches
- **CodeCop Compliance**: Only use BEGIN..END when the case branch contains multiple statements
- **Alternative Approach**: Use BEGIN..END consistently for visual alignment and easier future modification

**CodeCop Compliant:**
Only use BEGIN..END when case branches contain multiple statements, keeping simple branches without block structure.

**Alternative Approach:**
Use BEGIN..END consistently across all branches for visual alignment and easier future modification.

### Team Decision Framework

Consider these factors when choosing your approach:
- **Future Modification**: Consistent BEGIN..END makes adding statements to branches easier
- **Code Density**: CodeCop compliance reduces visual noise for simple case statements
- **Team Consistency**: Align with existing patterns in your codebase and team preferences
- **Maintenance Style**: Consider whether developers typically add logging or debugging to case branches

