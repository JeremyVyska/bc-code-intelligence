---
title: "AL Comment Spacing Standards"
domain: "roger-reviewer"
difficulty: "beginner"
bc_versions: "14+"
tags: ["comments", "documentation", "spacing"]
samples: "samples/al-comment-spacing-standards.md"
type: "improvement-pattern"
category: "code-formatting"
pattern_type: "good"
severity: "medium"
impact_level: "medium"
improvement_suggestion: "Apply consistent spacing around AL comments to improve readability and documentation clarity"
implementation_steps: ["standardize-comment-spacing", "apply-consistent-patterns", "validate-readability"]
validation_criteria: ["consistent-comment-spacing", "clear-documentation-separation"]
technical_areas: ["al-code"]
---

# AL Comment Spacing Standards

## Overview

Consistent spacing around AL comments improves code readability and ensures documentation integrates smoothly with code structure. Proper comment spacing helps developers distinguish between code and documentation while maintaining clean visual organization.

**Key Principle**: Use consistent spacing patterns around comments to create clear separation between documentation and executable code.

## Strategic Framework

### Documentation Integration
- **Visual Separation**: Clear spacing between comments and code elements
- **Consistency Patterns**: Uniform spacing application across all AL objects
- **Readability Focus**: Spacing that enhances documentation effectiveness
- **Code Organization**: Comments as integral part of code structure

### Communication Enhancement
Well-spaced comments improve developer communication by making documentation more accessible and visually integrated with code logic.

## Architecture Focus

### Comment Types
- **Line Comments**: Single-line documentation with appropriate spacing
- **Block Comments**: Multi-line documentation with consistent formatting
- **Inline Comments**: End-of-line explanations with proper separation
- **Section Headers**: Major section documentation with clear boundaries

### Spacing Patterns
Establish consistent spacing rules that work well with AL syntax highlighting and support effective documentation practices within development teams.

## Best Practices

### Spacing Guidelines
- **Blank line before** major comment blocks
- **Single space after** comment delimiters (//)
- **Consistent indentation** for comment content alignment
- **Appropriate separation** between comments and related code
- **Clear boundaries** around documentation sections

### Comment Organization
Group related comments effectively, maintain consistent formatting within comment blocks, and use spacing to create logical documentation structure.

### Integration with Code
Position comments to enhance code understanding without disrupting visual code flow. Use spacing to create natural reading rhythm between documentation and implementation.

## Anti-Patterns

### Avoid These Approaches
- **Inconsistent spacing** around comment delimiters
- **Cramped comments** without adequate separation from code
- **Excessive spacing** that fragments code unnecessarily
- **Poor indentation** in multi-line comment blocks
- **Random comment placement** without logical organization

Poor comment spacing makes documentation harder to read and reduces the effectiveness of code documentation efforts.

*Implementation examples: see samples/al-comment-spacing-standards.md*
*Related patterns: al-blank-line-organization.md, al-readability-optimization.md*