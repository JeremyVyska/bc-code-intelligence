---
title: "No. Series Validation Module Patterns (BC24+)"
domain: "dean-debug"
difficulty: "intermediate"
bc_versions: "24+"
tags: ["no-series-validation", "business-foundation", "module-validation", "bc24-migration", "data-integrity"]
prerequisites: ["no-series-module-patterns-bc24", "validation-frameworks", "business-foundation-architecture"]
samples: "samples/no-series-validation-module-bc24.md"
related_topics: ["no-series-module-patterns-bc24", "no-series-validation-patterns", "bc24-migration-patterns"]
---

# No. Series Validation Module Patterns (BC24+)

## Overview

The new No. Series module in BC24+ provides enhanced validation capabilities through the Business Foundation layer. These patterns leverage the improved validation methods and modular architecture to ensure number sequence integrity with better performance and clearer error handling.

**Key Benefit**: Utilizes the new No. Series module's built-in validation capabilities for more reliable and maintainable number sequence validation with enhanced error reporting.

## New Validation Architecture

### Business Foundation Integration
The No. Series module provides native validation methods that integrate seamlessly with the Business Foundation layer's error handling and logging systems.

### Enhanced Validation Methods
New dedicated validation methods replace complex custom validation logic:
- Built-in format validation
- Automatic relationship validation
- Improved error messaging and diagnostics

### Modular Validation Design
Validation is now distributed across focused codeunits rather than centralized in legacy NoSeriesManagement.

## Essential Validation Patterns

### Number Format Validation
**Legacy Pattern (BC14-23)**:
```al
// Custom validation logic required
if not NoSeriesMgt.CheckFormat("No.", "No. Series") then
    Error('Invalid format');
```

**New Pattern (BC24+)**:
```al
if not NoSeries.IsValidNo("No.", "No. Series") then
    Error('Number %1 is not valid for series %2', "No.", "No. Series");
```

### Series Existence Validation
**New Pattern (BC24+)**:
```al
NoSeries.VerifySeriesExists("No. Series");
```

### Manual Number Validation
**Legacy Pattern**:
```al
NoSeriesMgt.TestManual("No. Series");
if not NoSeriesMgt.ManualNoAllowed("No. Series") then
    Error('Manual numbers not allowed');
```

**New Pattern**:
```al
NoSeries.TestManual("No. Series");
// Built-in error handling provides clear messaging
```

### Series Relationship Validation
**New Pattern (BC24+)**:
```al
if not NoSeries.AreRelated(CurrentSeries, NewSeries) then
    Error('Series %1 and %2 are not related', CurrentSeries, NewSeries);
```

## Advanced Validation Integration

### Batch Validation
Utilize NoSeriesBatch for validating multiple number allocations:
```al
// Validate batch allocation feasibility
if not NoSeriesBatch.CanAllocateNumbers("No. Series", RequiredCount) then
    Error('Cannot allocate %1 numbers from series %2', RequiredCount, "No. Series");
```

### Date-Based Validation
Leverage enhanced date validation in the new module:
```al
if not NoSeries.IsValidForDate("No. Series", "Posting Date") then
    Error('Series %1 is not valid for date %2', "No. Series", "Posting Date");
```

### Cross-Series Validation
```al
// Validate consistency across related series
NoSeries.ValidateRelatedSeries("Primary Series", "Related Series");
```

## Migration from Legacy Validation

### Obsolete Validation Patterns
**Remove Legacy Patterns**:
- Custom format checking logic
- Manual series relationship validation
- Complex date range validation
- Direct table validation queries

**Replace with Module Methods**:
- Use built-in `IsValidNo()` for format validation
- Use `AreRelated()` for relationship checking
- Use `TestManual()` for manual number validation
- Use `VerifySeriesExists()` for existence checking

### Event Migration Strategy
**Obsolete Events** (no direct replacement):
- OnBeforeGetNextNo events
- OnAfterGetNextNo events
- Custom validation events in NoSeriesManagement

**New Approach**:
```al
// Implement validation through direct method calls
// Use error handling for validation failures
// Leverage module's built-in logging capabilities
```

## Enhanced Error Handling

### Improved Error Messages
The new module provides more descriptive error messages with context:
```al
// Automatic error context from module
NoSeries.GetNextNo("No. Series", WorkDate());
// Provides detailed error if series is invalid, exhausted, or misconfigured
```

### Structured Exception Handling
```al
if not NoSeries.TryGetNextNo(NextNo, "No. Series", WorkDate()) then begin
    // Handle specific validation failure scenarios
    HandleNumberGenerationError("No. Series");
end;
```

### Validation Result Processing
```al
ValidationResult := NoSeries.ValidateSeriesConfiguration("No. Series");
if not ValidationResult.Success then
    ProcessValidationErrors(ValidationResult);
```

## Performance-Optimized Validation

### Reduced Database Queries
The new module optimizes validation with fewer database round-trips:
```al
// Single call validates multiple aspects
if NoSeries.IsReadyForGeneration("No. Series", WorkDate()) then
    ProcessNumberGeneration();
```

### Cached Validation Results
```al
// Module handles caching internally
// Repeated validation calls are optimized
for i := 1 to DocumentCount do
    NoSeries.GetNextNo("No. Series", WorkDate()); // Cached validation
```

### Batch Validation Optimization
```al
// Validate entire batch before processing
ValidationResults := NoSeriesBatch.ValidateBatchAllocation("No. Series", BatchSize);
if ValidationResults.AllValid then
    ProcessBatch();
```

## Integration with Business Processes

### Transaction Validation
```al
// Validate number series within transaction context
NoSeries.ValidateForTransaction("No. Series", TransactionDate);
```

### Multi-Company Validation
```al
// Enhanced support for multi-company scenarios
NoSeries.ValidateAcrossCompanies("No. Series", CompanyList);
```

### Audit Trail Integration
```al
// Built-in audit trail support
NoSeries.LogValidationActivity("No. Series", ValidationAction::Validate);
```

## Best Practices for BC24+

### Leverage Built-In Validation
- Use module's native validation methods instead of custom logic
- Trust the module's error handling and messaging
- Utilize built-in logging and audit capabilities

### Simplify Validation Logic
- Remove complex custom validation patterns
- Focus on business-specific validation requirements
- Let the module handle technical validation aspects

### Error Handling Strategy
- Implement graceful degradation for validation failures
- Provide business-friendly error messages
- Use try-methods for scenarios where validation failure is expected

## Common Migration Pitfalls

### Over-Validation
Avoid duplicating validation that the new module already performs internally.

### Event Dependency
Don't rely on obsolete events - redesign validation logic using direct method calls.

### Custom Table Access
Avoid direct table manipulation - use module methods for all series operations.

## Testing Validation Migration

### Functional Validation Testing
- Verify all validation scenarios work with new module methods
- Test error handling and messaging improvements
- Validate performance improvements in validation-heavy scenarios

### Integration Testing
- Test validation integration with existing business processes
- Verify compatibility with custom validation requirements
- Test cross-series validation scenarios

### Regression Testing
- Ensure existing validation behavior is preserved
- Test edge cases and error conditions
- Validate performance under load

*Complete BC24+ validation examples: samples/no-series-validation-module-bc24.md*
*Core module patterns: no-series-module-patterns-bc24.md*
*Migration guidance: bc24-no-series-conversion-guide.md*