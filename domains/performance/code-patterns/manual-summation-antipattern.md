---
title: "Manual Summation Anti-Pattern"
domain: performance
type: code-pattern
pattern_type: bad
category: performance
severity: high
name: manual-summation-instead-of-sift
regex_patterns:
  - "repeat\\s+[\\s\\S]*?\\+=[\\s\\S]*?until.*\\.Next\\(\\)"
  - "while.*\\.Next\\(\\)\\s*=\\s*0[\\s\\S]*?\\+="
description: "Manual record summation detected - consider using SIFT CalcSums for better performance"
related_topics:
  - sift-technology-fundamentals
  - query-performance-patterns
  - flowfield-optimization
replacement_suggestion: "Use CalcSums() with appropriate SIFT index"
domains: [performance, data-access]
difficulty: intermediate
tags: [anti-pattern, sift, performance, summation]
detection_confidence: high
impact_level: high
---

# Manual Summation Anti-Pattern

## Problem
Manual loops through records to calculate sums create significant performance overhead and don't leverage SQL Server's optimized aggregation capabilities.

## Bad Example
```al
TotalAmount := 0;
SalesLine.SetRange("Document No.", SalesHeader."No.");
if SalesLine.FindSet() then
    repeat
        TotalAmount += SalesLine.Amount;
    until SalesLine.Next() = 0;
```

## Good Solution
```al
SalesLine.SetRange("Document No.", SalesHeader."No.");
SalesLine.CalcSums(Amount);
TotalAmount := SalesLine.Amount;
```

## Why This Matters
- **Performance**: CalcSums uses SIFT technology for sub-second aggregation
- **Scalability**: Manual loops degrade linearly with record count
- **Memory**: Avoids loading unnecessary record data
- **Maintainability**: Single line vs complex loop logic

## Related Patterns
- Use `SetLoadFields()` before summation if you need other fields
- Ensure SIFT indexes exist for the fields being summed
- Consider `CalcFields()` for FlowFields when appropriate