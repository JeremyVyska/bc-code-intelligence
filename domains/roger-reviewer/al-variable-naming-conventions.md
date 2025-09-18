---
title: "AL Variable Naming Conventions"
domain: "roger-reviewer"
difficulty: "beginner"
bc_versions: "14+"
tags: ["variables", "naming", "code-style", "readability"]
type: "improvement-pattern"
category: "code-quality"
pattern_type: "good"
severity: "medium"
impact_level: "medium"
improvement_suggestion: "Apply consistent variable naming conventions to enhance code readability and maintainability"
implementation_steps: ["establish-naming-standards", "apply-consistent-patterns", "validate-conventions"]
validation_criteria: ["consistent-naming-application", "improved-readability"]
technical_areas: ["al-code"]
---

# AL Variable Naming Conventions

## Overview

Consistent variable naming in AL code improves readability, maintainability, and team collaboration. Business Central development follows Microsoft's naming conventions with specific adaptations for ERP domain concepts.

**Key Principle**: Variable names should clearly communicate purpose, scope, and data type without requiring additional documentation.

## Strategic Framework

### Naming Pattern Hierarchy
- **Local Variables**: CamelCase starting with lowercase (customerRecord, totalAmount)
- **Global Variables**: CamelCase starting with uppercase (CustomerBuffer, TotalSalesAmount)
- **Constants**: ALL_CAPS with underscores (MAX_RETRY_COUNT, DEFAULT_TIMEOUT)
- **Temporary Variables**: Prefix with "Temp" (TempCustomer, TempItemLedgerEntry)

### Domain-Specific Patterns
- **Record Variables**: Use table name without "Record" suffix (Customer, not CustomerRecord)
- **Buffer Variables**: Append "Buffer" for temporary storage (CustomerBuffer)
- **Counter Variables**: Use descriptive names (CustomerCount, not i or counter)
- **Boolean Variables**: Use positive phrasing with "Is/Has/Can" prefixes (IsPosted, HasPermission)

## Architecture Focus

### Scope-Based Conventions
**Local Variables** follow function-specific naming that clearly indicates their role within the procedure context.

**Global Variables** use broader context names that reflect their application-wide purpose and lifetime.

**Parameter Variables** match their usage pattern - input parameters use descriptive names, output parameters often include "Result" or "Output" suffixes.

### Type-Specific Patterns
- **Codeunit Variables**: Include "Mgt" or "Management" suffix (CustomerMgt, SalesMgt)
- **Page Variables**: Use "Page" suffix when storing page references (CustomerPage)
- **Report Variables**: Use "Report" suffix for report references (SalesReport)
- **XMLport Variables**: Use "XMLport" suffix (CustomerXMLport)

## Best Practices

### Clarity and Context
- **Avoid Abbreviations**: Use complete words unless standard BC abbreviations (Qty, Amt, No.)
- **Include Units**: For measurements, include unit context (AmountLCY, QtyBase)
- **Specify Direction**: For parameters, indicate flow (CustomerNoIn, ResultOut)
- **Use Business Terms**: Prefer domain language over technical terms (PostingDate vs DateVar)

### Consistency Patterns
- **Maintain Team Standards**: Follow established project conventions over personal preferences
- **Use Standard Prefixes**: Apply consistent prefixes for similar variable types
- **Document Deviations**: When breaking convention for specific reasons, include inline comments
- **Review and Refactor**: Regular code reviews should enforce naming consistency

### Hungarian Notation Avoidance
Modern AL development avoids Hungarian notation. Variable types are clear from declaration context, making type prefixes unnecessary and potentially misleading during refactoring.

## Anti-Patterns

### Avoid These Approaches
- **Generic Names**: Avoid "Rec", "TempRec", "Buffer" without context
- **Misleading Abbreviations**: Don't create custom abbreviations (CustAddr vs CustomerAddress)
- **Type Prefixes**: Avoid Hungarian notation (strName, intCount, bolIsValid)
- **Inconsistent Casing**: Mixed casing patterns within same codebase

Poor variable naming creates maintenance problems and makes code harder to understand, review, and modify effectively.

*Related patterns: al-variable-declaration-order.md, al-standard-abbreviations.md*