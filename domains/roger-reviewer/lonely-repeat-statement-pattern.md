---
title: "Lonely Repeat Statement Pattern"
domain: "roger-reviewer"
difficulty: "beginner"
bc_versions: "14+"
tags: ["code-formatting", "repeat-statement", "readability", "code-standards"]
prerequisites: ["al-syntax-basics", "control-structures"]
samples: "samples/lonely-repeat-examples.md"
---

# Lonely Repeat Statement Pattern

## Overview

The "lonely repeat" anti-pattern occurs when a `repeat` keyword appears isolated on its own line without the associated `until` condition being visually connected. This formatting choice reduces code readability and violates AL formatting conventions.

**Core Issue**: Isolated `repeat` statements create visual disconnection from their terminating conditions, making control flow harder to follow.

## Pattern Definition

### Lonely Repeat Structure
```
// Anti-pattern: Lonely repeat
repeat
    // statements
until condition;
```

**Problem**: The `repeat` keyword lacks visual connection to its scope and terminating condition.

### Preferred Formatting
```
// Preferred: Connected repeat block
repeat
    // statements
until condition;
```

**Solution**: Maintain visual connection between `repeat`, statement block, and `until` condition through consistent indentation and spacing.

## Formatting Standards

### Standard AL Formatting Rules
- **Consistent indentation**: `repeat` aligns with containing scope
- **Statement block indentation**: Contents indented one level from `repeat`
- **Until alignment**: `until` aligns with `repeat` keyword
- **Condition formatting**: Condition on same line as `until` when concise

### Visual Readability Principles
- **Scope boundaries**: Clear visual indication of repeat block extent
- **Flow connection**: Visual link between control keyword and condition
- **Consistent spacing**: Uniform spacing patterns across all repeat statements

## Common Violations

```
// Violation: Extra blank lines and misaligned until
repeat

    ProcessRecord();
    until EOF;

// Violation: Complex condition on same line
repeat
    ProcessRecord();
until (Counter > MaxRecords) and (not ErrorOccurred) and ValidateData();
```

**Problems**: Unnecessary blank lines break visual flow, misaligned `until` breaks connection with `repeat` scope, and complex conditions reduce readability.

## Best Practice Patterns

### Simple Repeat Blocks
```
// Best practice: Clean, aligned structure
repeat
    ProcessNextRecord();
    Counter += 1;
until (Counter > MaxRecords) or EOF;
```

### Complex Condition Formatting
```
// Best practice: Multi-line complex conditions
repeat
    ProcessRecord();
    ValidateData();
until (Counter > MaxRecords) and
     (not ErrorOccurred) and
     ValidateCompleted;
```

### Nested Repeat Handling
```
// Best practice: Clear nesting structure
repeat
    repeat
        ProcessSubRecord();
    until SubRecordComplete;
    FinalizeRecord();
until AllRecordsProcessed;
```

## Implementation

Use AL formatting tools for automated detection and establish team formatting standards with code review enforcement.

*Complete formatting examples: samples/lonely-repeat-examples.md*
*AL formatting standards: roger-reviewer-index.json#formatting-rules*
*Automated formatting setup: sam-coder-index.json#formatting-configuration*