---
title: "AL Keyword Indentation Rules"
domain: "roger-reviewer"
difficulty: "intermediate"
bc_versions: "14+"
tags: ["indentation", "keywords", "code-structure"]
samples: "samples/al-keyword-indentation-rules.md"
type: "improvement-pattern"
category: "code-formatting"
pattern_type: "good"
severity: "medium"
impact_level: "medium"
improvement_suggestion: "Apply consistent indentation rules for AL keywords to create clear code hierarchy"
implementation_steps: ["define-indentation-standards", "apply-keyword-rules", "validate-consistency"]
validation_criteria: ["consistent-keyword-indentation", "clear-code-hierarchy"]
technical_areas: ["al-code"]
---

# AL Keyword Indentation Rules

## Overview

Consistent indentation rules for AL keywords create clear visual hierarchy in code structure, making it easier to understand control flow, scope boundaries, and logical organization. Proper keyword indentation is fundamental to readable and maintainable AL code.

**Key Principle**: Keywords should follow consistent indentation patterns that reflect code structure and logical hierarchy.

## Strategic Framework

### Indentation Hierarchy
- **Block Structure**: Keywords reflect nesting levels and scope boundaries
- **Visual Clarity**: Indentation immediately shows code organization
- **Consistency Standards**: Uniform application across all AL objects
- **Readability Focus**: Indentation supports quick code comprehension

### Code Structure Reflection
Keyword indentation should mirror the logical structure of AL code, making scope boundaries and control flow patterns immediately visible to developers.

## Architecture Focus

### Keyword Categories
- **Control Flow Keywords**: if, else, while, for, case statements
- **Block Delimiters**: begin, end pairs and scope boundaries
- **Declaration Keywords**: var, procedure, trigger declarations
- **Object Keywords**: table, page, codeunit structure elements

### Hierarchy Patterns
Establish clear indentation increments that consistently reflect nesting levels and logical code organization. Standard indentation helps teams maintain consistent code appearance.

## Best Practices

### Indentation Standards
- **Consistent increment size** (typically 4 spaces) for each nesting level
- **Aligned block keywords** (begin/end pairs at same indentation)
- **Control flow alignment** showing decision structure clearly
- **Scope reflection** through indentation hierarchy
- **Declaration organization** with appropriate keyword positioning

### Consistency Application
Apply indentation rules uniformly across all AL objects, maintain consistent patterns within teams, and use automated formatting tools where possible to ensure compliance.

### Readability Enhancement
Structure keyword indentation to support quick visual parsing of code organization. Consistent indentation reduces cognitive load when reviewing complex AL procedures.

## Anti-Patterns

### Avoid These Approaches
- **Inconsistent indentation increments** creating visual confusion
- **Misaligned begin/end pairs** obscuring block boundaries
- **Random keyword positioning** without logical organization
- **Mixed indentation styles** within single AL object
- **Excessive indentation** making code difficult to follow

Poor keyword indentation makes code structure unclear and increases the difficulty of understanding control flow and scope relationships.

*Implementation examples: see samples/al-keyword-indentation-rules.md*
*Related patterns: al-line-start-keyword-positioning.md, al-case-action-formatting.md*