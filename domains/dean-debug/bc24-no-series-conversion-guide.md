---
title: "BC24 No. Series Conversion Guide"
domain: "dean-debug"
difficulty: "advanced"
bc_versions: "24+"
tags: ["bc24-migration", "no-series-conversion", "legacy-modernization", "breaking-changes", "conversion-guide"]
prerequisites: ["no-series-implementation-patterns", "no-series-module-patterns-bc24", "business-foundation-architecture"]
samples: "samples/bc24-no-series-conversion.md"
related_topics: ["no-series-module-patterns-bc24", "no-series-validation-module-bc24", "no-series-implementation-patterns"]
---

# BC24 No. Series Conversion Guide

## Overview

This guide provides comprehensive step-by-step instructions for converting legacy No. Series implementations (BC14-23) to the new Business Foundation module architecture (BC24+). It addresses Microsoft's official obsoletions and provides practical conversion strategies.

**Critical Timeline**: NoSeriesManagement (Codeunit 396) is obsolete in BC24.0 - immediate conversion required for compatibility.

## Conversion Overview

### What Changed in BC24
- **NoSeriesManagement (Codeunit 396)**: Completely obsolete
- **New Architecture**: Business Foundation modular design
- **New Codeunits**: No. Series (310), NoSeriesBatch
- **Table Changes**: Sales/Purchase series tables merged into generic No. Series Line
- **Method Changes**: Simplified parameter structures and clearer method names

### Conversion Scope Assessment
Before starting conversion, inventory your codebase:

1. **Search Patterns**: `NoSeriesMgt`, `NoSeriesManagement`
2. **Method Usage**: `InitSeries`, `GetNextNo`, `TestManual`, `SelectSeries`
3. **Event Subscriptions**: Any events on obsolete codeunit
4. **Custom Extensions**: Extensions of NoSeriesManagement
5. **Table References**: Direct access to No. Series tables

## Step-by-Step Conversion Process

### Step 1: Inventory and Planning

#### Code Discovery
```al
// Search your codebase for these patterns:
// 1. Variable declarations
var NoSeriesMgt: Codeunit NoSeriesManagement;

// 2. Method calls
NoSeriesMgt.InitSeries(...)
NoSeriesMgt.GetNextNo(...)
NoSeriesMgt.TestManual(...)
NoSeriesMgt.SelectSeries(...)

// 3. Event subscriptions
[EventSubscriber(ObjectType::Codeunit, Codeunit::NoSeriesManagement, 'OnBeforeGetNextNo', '', true, true)]
```

#### Create Conversion Matrix
| Legacy Method | New Method | Conversion Notes |
|---------------|------------|------------------|
| `InitSeries()` | `GetNextNo()` | Simplified parameters |
| `GetNextNo(..., false)` | `PeekNextNo()` | Preview only, doesn't consume |
| `GetNextNo(..., true)` | `GetNextNo()` | Allocates and consumes number |
| `TestManual()` | `TestManual()` | Same name, new codeunit |
| `SelectSeries()` | `LookupRelatedNoSeries()` | Different parameter structure |

### Step 2: Core Method Conversions

#### InitSeries Conversion
**Before (BC14-23)**:
```al
var
    NoSeriesMgt: Codeunit NoSeriesManagement;
begin
    NoSeriesMgt.InitSeries(GetNoSeriesCode(), xRec."No. Series", 0D, "No.", "No. Series");
end;
```

**After (BC24+)**:
```al
var
    NoSeries: Codeunit "No. Series";
begin
    if "No." = '' then
        "No." := NoSeries.GetNextNo("No. Series", WorkDate());
end;
```

#### GetNextNo Conversion
**Before**:
```al
NextNo := NoSeriesMgt.GetNextNo("No. Series", WorkDate(), false); // Peek only
ActualNo := NoSeriesMgt.GetNextNo("No. Series", WorkDate(), true); // Allocate
```

**After**:
```al
NextNo := NoSeries.PeekNextNo("No. Series", WorkDate()); // Peek only
ActualNo := NoSeries.GetNextNo("No. Series", WorkDate()); // Allocate
```

#### When to Use GetNextNo vs PeekNextNo

**Use `PeekNextNo()` for**:
- **UI Display**: Showing next number in forms before user commits
- **Preview Operations**: Preview posting scenarios
- **Validation Logic**: Checking number availability without consuming
- **Form Initialization**: Displaying what number would be assigned
- **Testing/Diagnostics**: Verifying series configuration

**Use `GetNextNo()` for**:
- **Document Creation**: Actually creating/posting business documents
- **Final Assignment**: Committing to use the number permanently
- **Transaction Completion**: When business logic is ready to consume the number
- **Batch Operations**: Allocating numbers for confirmed processing

**Decision Framework**:
```al
// Ask yourself: "Am I ready to permanently use this number?"
if IsPreviewMode or IsDisplayOnly or IsValidationCheck then
    NextNo := NoSeries.PeekNextNo("No. Series", WorkDate())
else
    NextNo := NoSeries.GetNextNo("No. Series", WorkDate());
```

#### Migration Opportunity: Add Preview Posting Support

**Legacy Pattern** (many old implementations):
```al
// Old code often only supported final posting
procedure PostDocument()
begin
    "No." := NoSeriesMgt.GetNextNo("No. Series", WorkDate(), true);
    // ... posting logic
end;
```

**Enhanced Pattern** (BC24+ with preview support):
```al
procedure PostDocument(Preview: Boolean)
begin
    if Preview then
        "No." := NoSeries.PeekNextNo("No. Series", WorkDate())
    else
        "No." := NoSeries.GetNextNo("No. Series", WorkDate());
    // ... posting logic
end;

procedure PreviewPosting()
begin
    PostDocument(true); // Uses PeekNextNo
end;

procedure FinalPosting()
begin
    PostDocument(false); // Uses GetNextNo
end;
```

**Modernization Questions to Ask**:
- Does this posting routine support preview mode?
- Could users benefit from seeing the assigned number before committing?
- Is there a "Preview Posting" action that should show the next number?
- Would preview posting improve user experience in this scenario?

#### Manual Testing Conversion
**Before**:
```al
NoSeriesMgt.TestManual("No. Series");
if NoSeriesMgt.ManualNoAllowed("No. Series") then
    AllowManual := true;
```

**After**:
```al
NoSeries.TestManual("No. Series"); // Built-in error handling
// ManualNoAllowed is replaced by TestManual with automatic error
```

#### Series Selection Conversion
**Before**:
```al
NoSeriesMgt.SelectSeries("No. Series", xRec."No. Series", "No. Series");
```

**After**:
```al
NoSeries.LookupRelatedNoSeries("No. Series", "No. Series");
```

### Step 3: Advanced Pattern Conversions

#### Batch Operations
**New Capability (BC24+)**:
```al
var
    NoSeriesBatch: Codeunit "No. Series - Batch";
    NumberList: List of [Code[20]];
begin
    NumberList := NoSeriesBatch.GetNextNos("No. Series", WorkDate(), BatchSize);
end;
```

#### Series Relationship Handling
**New Pattern (BC24+)**:
```al
if NoSeries.AreRelated("Primary Series", "Secondary Series") then
    // Handle related series logic
    ProcessRelatedSeries();
```

### Step 4: Event Migration Strategy

#### Obsolete Events (No Direct Replacement)
These events are removed in BC24 without direct replacement:
```al
// OBSOLETE - Remove these event subscriptions
[EventSubscriber(ObjectType::Codeunit, Codeunit::NoSeriesManagement, 'OnBeforeGetNextNo', '', true, true)]
[EventSubscriber(ObjectType::Codeunit, Codeunit::NoSeriesManagement, 'OnAfterGetNextNo', '', true, true)]
```

#### Alternative Approaches
```al
// Replace event-driven logic with direct method calls
procedure CustomGetNextNo(SeriesCode: Code[20]): Code[20]
var
    NoSeries: Codeunit "No. Series";
    NextNo: Code[20];
begin
    // Pre-processing logic (replaces OnBeforeGetNextNo)
    PreProcessNumberGeneration(SeriesCode);

    // Get number
    NextNo := NoSeries.GetNextNo(SeriesCode, WorkDate());

    // Post-processing logic (replaces OnAfterGetNextNo)
    PostProcessNumberGeneration(SeriesCode, NextNo);

    exit(NextNo);
end;
```

### Step 5: Error Handling Migration

#### Enhanced Error Messages
**Before**:
```al
if not NoSeriesMgt.TestSeries("No. Series") then
    Error('Series validation failed');
```

**After**:
```al
// Built-in descriptive error messages
NoSeries.GetNextNo("No. Series", WorkDate()); // Automatic detailed errors
```

#### Try-Method Pattern
**New Approach (BC24+)**:
```al
if NoSeries.TryGetNextNo(NextNo, "No. Series", WorkDate()) then
    ProcessSuccessfulGeneration(NextNo)
else
    HandleGenerationFailure("No. Series");
```

### Step 6: Performance Optimization

#### Reduced Object Coupling
**Before**: Complex dependencies with NoSeriesManagement
**After**: Clean dependencies on focused No. Series module

#### Optimized Database Access
The new module provides better performance through:
- Reduced database queries
- Improved caching mechanisms
- Better concurrency handling

### Step 7: Testing Strategy

#### Functional Testing Checklist
- [ ] All number generation scenarios work correctly
- [ ] Manual number entry functions properly
- [ ] Series relationships are maintained
- [ ] Date-based generation works as expected
- [ ] Error handling provides appropriate messages

#### Performance Testing
- [ ] Number generation performance meets or exceeds legacy performance
- [ ] High-concurrency scenarios perform adequately
- [ ] Batch operations show expected performance improvements

#### Regression Testing
- [ ] Existing number sequences continue to work
- [ ] No gaps or duplicates in number generation
- [ ] Business process integration remains functional

## Common Conversion Challenges

### Challenge 1: Event Dependencies
**Problem**: Code relies on obsolete events
**Solution**: Redesign logic using direct method calls and procedure composition

### Challenge 2: Parameter Mapping
**Problem**: New methods have different parameter structures
**Solution**: Create wrapper procedures during transition period

### Challenge 3: Custom Extensions
**Problem**: Extensions of NoSeriesManagement no longer work
**Solution**: Implement custom logic using the new module's extensibility points

### Challenge 4: Complex Validation Logic
**Problem**: Custom validation tied to obsolete methods
**Solution**: Leverage new module's built-in validation capabilities

## Conversion Validation

### Pre-Conversion Checklist
- [ ] Complete inventory of NoSeriesManagement usage
- [ ] Backup of current implementation
- [ ] Test plan for all affected scenarios
- [ ] Rollback strategy prepared

### Post-Conversion Checklist
- [ ] All obsolete references removed
- [ ] New module methods properly implemented
- [ ] Error handling provides appropriate feedback
- [ ] Performance meets expectations
- [ ] Documentation updated

### Production Deployment Checklist
- [ ] Comprehensive testing in sandbox environment
- [ ] User acceptance testing completed
- [ ] Deployment runbook prepared
- [ ] Rollback procedures tested

## Best Practices

### During Conversion
- Convert one method at a time to minimize risk
- Test thoroughly after each conversion step
- Maintain parallel implementations during transition if needed
- Document all changes for team knowledge transfer

### Post-Conversion
- Leverage new module capabilities for enhanced functionality
- Simplify validation logic using built-in methods
- Monitor performance improvements
- Plan for future module enhancements

### Team Coordination
- Establish conversion standards across development team
- Create shared conversion utilities and wrapper procedures
- Coordinate timing of conversion across related projects
- Share lessons learned and best practices

*Complete conversion examples: samples/bc24-no-series-conversion.md*
*BC24+ implementation patterns: no-series-module-patterns-bc24.md*
*BC24+ validation patterns: no-series-validation-module-bc24.md*
*Legacy patterns (reference): no-series-implementation-patterns.md*