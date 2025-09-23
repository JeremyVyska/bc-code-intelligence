---
title: "AL File Naming Conventions"
domain: ["sam-coder", "roger-reviewer"]
difficulty: "beginner"
bc_versions: "14+"
tags: ["file-naming", "conventions", "code-style", "organization"]
type: "improvement-pattern"
category: "code-organization"
pattern_type: "good"
severity: "high"
impact_level: "medium"
improvement_suggestion: "Apply consistent AL file naming conventions following alguidelines.dev standards"
implementation_steps: ["identify-object-name", "apply-object-type-suffix", "validate-naming-pattern"]
validation_criteria: ["consistent-naming-application", "standards-compliance"]
technical_areas: ["al-code", "file-organization"]
related_topics: ["al-variable-naming-conventions", "al-standard-abbreviations"]
---

# AL File Naming Conventions

## Overview

AL file naming conventions ensure consistent project organization and improve developer experience. Following standardized patterns makes codebases more maintainable and professional.

**Source**: Based on [alguidelines.dev Rule #2: File Naming Conventions](https://alguidelines.dev/docs/vibe-coding/al-naming-conventions/)

**Key Principle**: File names should be descriptive and match the AL object name within the files using the pattern `<ObjectName>.<ObjectType>.al`.

## Standard File Naming Pattern

### Core Pattern: `<ObjectName>.<ObjectType>.al`

**Table Extensions:**
```
✅ CORRECT: CustomerExt.TableExt.al
✅ CORRECT: SalesHeaderExt.TableExt.al
❌ WRONG: Tab-Ext50100.CustomerExt.al
❌ WRONG: TableExt50100.CustomerExt.al
```

**Page Extensions:**
```
✅ CORRECT: CustomerCardExt.PageExt.al
✅ CORRECT: InventorySetupExt.PageExt.al
❌ WRONG: Pag-Ext50100.CustomerCardExt.al
❌ WRONG: PageExt50100.CustomerCardExt.al
```

**Codeunits:**
```
✅ CORRECT: CustomerCommentsManager.Codeunit.al
✅ CORRECT: PostSalesInvoice.Codeunit.al
❌ WRONG: Cod50100.CustomerCommentsManager.al
❌ WRONG: Codeunit50100.CustomerCommentsManager.al
```

**Event Subscribers:**
```
✅ CORRECT: SalesEventsSubscriber.Codeunit.al
✅ CORRECT: CustomerEventsHandler.Codeunit.al
```

## Object Type Suffixes

### Standard Suffixes
- **Tables**: `.Table.al`
- **Table Extensions**: `.TableExt.al`
- **Pages**: `.Page.al`
- **Page Extensions**: `.PageExt.al`
- **Page Customizations**: `.PageCust.al`
- **Reports**: `.Report.al`
- **Report Extensions**: `.ReportExt.al`
- **Codeunits**: `.Codeunit.al`
- **Queries**: `.Query.al`
- **XMLports**: `.XMLport.al`
- **Enums**: `.Enum.al`
- **Enum Extensions**: `.EnumExt.al`
- **Interfaces**: `.Interface.al`
- **Control Add-ins**: `.ControlAddin.al`
- **Dotnet**: `.Dotnet.al`
- **Profiles**: `.Profile.al`
- **Permission Sets**: `.PermissionSet.al`
- **Permission Set Extensions**: `.PermissionSetExt.al`

## Best Practices

### Object Name Guidelines
- **Use Pascal Case**: CustomerCommentsManager (not customercommentsmanager)
- **Be Descriptive**: Clearly indicate object purpose
- **Avoid Abbreviations**: Use full words unless standard BC terms
- **Include Context**: For extensions, indicate what you're extending

### Organization Patterns
- **Logical Grouping**: Group related objects by business domain
- **Consistent Prefixes**: Use company/solution prefixes consistently
- **Module Structure**: Organize files into logical modules or folders

### Extension-Specific Naming
- **Table Extensions**: `[TableName]Ext.TableExt.al`
- **Page Extensions**: `[PageName]Ext.PageExt.al`
- **Clear Extension Purpose**: When extending for specific features, include context

## Anti-Patterns to Avoid

### Legacy/Incorrect Patterns
```
❌ Tab-Ext50000.CustomerExt.al          // Old NAV-style naming
❌ TableExt50000.CustomerExt.al         // Object ID in filename
❌ CustomerExt-50000.al                 // Mixed patterns
❌ customer_ext.table_ext.al            // Underscore notation
❌ CustomerExtension.TableExtension.al  // Verbose suffixes
```

### Why These Are Wrong
- **Object IDs in Filenames**: IDs change, names are stable
- **Abbreviated Prefixes**: "Tab-Ext" is non-standard and unclear
- **Inconsistent Casing**: Makes files hard to locate and sort
- **Verbose Extensions**: ".TableExtension" is unnecessarily long

## Implementation Strategy

### File Naming Checklist
1. **Identify Object Name**: Use the actual AL object name from inside the file
2. **Apply Correct Suffix**: Use standard `.ObjectType.al` pattern
3. **Validate Consistency**: Ensure all project files follow same convention
4. **Update References**: Check that any references or documentation align

### Refactoring Existing Projects
- **Rename Systematically**: Update all files to follow standard pattern
- **Update Git History**: Consider preserving file history during renames
- **Team Communication**: Ensure all team members understand new conventions
- **Tool Configuration**: Update any build scripts or tools that reference filenames

## Integration with Development Tools

### VS Code Integration
- **File Explorer Sorting**: Consistent naming improves file navigation
- **Search and Replace**: Standard patterns make project-wide changes easier
- **Extensions**: AL Language extension works better with standard naming

### Source Control Benefits
- **File Tracking**: Consistent naming improves change tracking
- **Merge Conflicts**: Reduced conflicts from naming inconsistencies
- **Review Process**: Easier code review with predictable file organization

*This is a foundational standard that affects all AL development. Proper file naming is the first step toward professional, maintainable Business Central extensions.*