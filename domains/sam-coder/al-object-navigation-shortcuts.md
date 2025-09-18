---
title: "AL Object Navigation Shortcuts"
domain: "sam-coder"
difficulty: "intermediate"
bc_versions: "14+"
tags: ["navigation", "al-objects", "workspace", "symbol-search", "dependencies"]
prerequisites: ["vs-code-al-shortcuts", "al-project-structure"]
---

# AL Object Navigation Shortcuts

## Overview

Efficient navigation between AL objects is crucial for understanding code relationships, debugging complex scenarios, and maintaining large AL projects. Master these navigation patterns to move fluidly through Business Central codebases.

**Core Concept**: AL object navigation combines symbol search, dependency tracking, and workspace awareness for rapid code exploration.

## Symbol-Based Navigation

### Go-To Definition Patterns
**F12** on any AL symbol provides immediate navigation:
- **Object references**: Jump from usage to object definition
- **Procedure calls**: Navigate to procedure implementation
- **Variable declarations**: Find where variables are declared
- **Field references**: Jump to table field definitions

### Peek Definition Workflow
**Alt+F12** opens inline preview without losing current context:
- **Quick reference checking**: Verify procedure signatures without navigation
- **Multi-object comparison**: Compare related objects side-by-side
- **Context preservation**: Maintain current editing position while exploring

### Reference Discovery
**Shift+F12** reveals all usages of current symbol:
- **Impact analysis**: Understand where objects/procedures are used
- **Refactoring planning**: Identify all affected code before changes
- **Documentation research**: Find usage examples across codebase

## Workspace-Level Navigation

### Global Symbol Search
**Ctrl+T** enables rapid workspace-wide object discovery:
- **Object name search**: Find tables, pages, codeunits by partial name
- **Procedure location**: Locate specific procedures across all objects
- **Fuzzy matching**: Use partial strings to find objects quickly

### AL-Specific Object Navigation
**Ctrl+Shift+P → "AL: Go!"** provides enhanced AL object search:
- **Object type filtering**: Search within specific object types (tables, pages)
- **Object ID navigation**: Jump directly to objects by ID number
- **Extension awareness**: Navigate across multiple AL extensions

### File-Level Symbol Navigation
**Ctrl+Shift+O** shows current file's symbol outline:
- **Procedure listing**: Quick access to all procedures in current object
- **Variable declarations**: Jump to variable declaration sections
- **Structured navigation**: Navigate by AL object structure (triggers, procedures)

## Dependency Tracking

### Cross-Object Relationships
Understanding AL object dependencies through navigation:
- **Table-Page relationships**: Navigate from table fields to related pages
- **Codeunit dependencies**: Track procedure calls across codeunits
- **Report-Dataset connections**: Follow data flow from datasets to layout

### Extension Boundary Navigation
Navigate between AL extensions and base application:
- **Base application references**: Jump to standard BC objects from extensions
- **Extension dependencies**: Navigate between dependent AL extensions
- **Symbol resolution**: Understand which extension provides specific symbols

## Advanced Navigation Techniques

### Multi-Cursor Object Exploration
Use **Ctrl+D** for simultaneous multi-object navigation:
- **Pattern identification**: Select similar symbols across multiple objects
- **Bulk navigation**: Navigate to multiple related definitions simultaneously
- **Comparison workflows**: Compare similar patterns across objects

### Breadcrumb Navigation
Leverage VS Code breadcrumbs for AL object context:
- **Object hierarchy**: Visual representation of current location in AL object
- **Quick parent navigation**: Click breadcrumbs to navigate to parent scopes
- **Context awareness**: Understand current position within complex AL objects

### Search Integration with Navigation
Combine **Ctrl+Shift+F** with navigation shortcuts:
- **Usage pattern search**: Find specific AL patterns across workspace
- **Cross-reference validation**: Verify object relationships through search
- **Documentation discovery**: Find comments and documentation related to objects

## Navigation Efficiency Patterns

### Context-Preserving Exploration
- **Tab management**: Use multiple tabs for related AL objects
- **Split editor**: Compare related objects side-by-side
- **Breadcrumb trails**: Maintain awareness of navigation history

### Rapid Discovery Workflows
- **Symbol-first navigation**: Start with symbol search, then explore context
- **Reference-driven exploration**: Use "Find All References" to understand object usage
- **Definition-to-usage flow**: Navigate from definitions to all usage contexts

### Team Navigation Standards
- **Consistent naming patterns**: Enable predictable object discovery
- **Documentation links**: Include navigation hints in object comments
- **Dependency documentation**: Maintain clear dependency maps for complex solutions

## Common Navigation Scenarios

### Debugging Complex Flows
Navigate efficiently through call stacks and data flows:
1. **Start at error location** using diagnostic navigation
2. **Trace back through call stack** using Go-to-Definition
3. **Identify data sources** through table/field navigation
4. **Verify related objects** using Find-All-References

### Code Review Navigation
Systematic exploration of code changes:
1. **Start with modified objects** in source control view
2. **Navigate to related objects** using dependency tracking
3. **Verify impact scope** using reference discovery
4. **Check integration points** through cross-object navigation

*Navigation practice exercises: samples/al-navigation-practice.md*
*Workspace search patterns: sam-coder-index.json#workspace-navigation*
*AL object relationships: code-design-index.json#object-dependencies*