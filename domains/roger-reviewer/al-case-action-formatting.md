---
title: "AL Case Action Formatting"
domain: "roger-reviewer"
difficulty: "intermediate"
bc_versions: "14+"
tags: ["case-statements", "formatting", "control-flow"]
samples: "samples/al-case-action-formatting.md"
type: "improvement-pattern"
category: "code-formatting"
pattern_type: "good"
severity: "medium"
impact_level: "medium"
improvement_suggestion: "Format case statement actions with consistent indentation and clear action boundaries"
implementation_steps: ["standardize-case-indentation", "align-action-blocks", "validate-readability"]
validation_criteria: ["consistent-indentation", "clear-action-separation"]
technical_areas: ["al-code"]
---

# AL Case Action Formatting

## Overview

Proper formatting of case statement actions in AL code ensures clear visual separation between different case branches and improves code readability. Consistent case action formatting helps developers quickly understand control flow logic and reduces errors in complex decision structures.

**Key Principle**: Each case action should be clearly separated and consistently indented to highlight the decision structure.

## Strategic Framework

### Case Statement Structure
- **Visual Hierarchy**: Clear indentation showing case relationship to parent statement
- **Action Boundaries**: Distinct separation between different case actions
- **Consistent Patterns**: Uniform formatting across all case statements in codebase
- **Readability Focus**: Format that supports quick visual parsing of decision logic

### Control Flow Clarity
Case formatting should make the decision structure immediately apparent to code reviewers and maintainers. Well-formatted case statements reduce cognitive overhead when understanding complex business logic.

## Architecture Focus

### Indentation Standards
- **Case Labels**: Consistent alignment with case keyword
- **Action Blocks**: Proper indentation for multi-line actions
- **Nested Structures**: Clear hierarchy for nested case statements
- **Code Organization**: Logical grouping of related case branches

### Visual Consistency
Establish consistent patterns for case action formatting that work well with AL language structure and team coding practices. Consistent formatting improves code review efficiency.

## Best Practices

### Formatting Patterns
- **Consistent indentation** for all case actions within statement
- **Clear separation** between case labels and actions
- **Aligned case labels** for easy visual scanning
- **Proper spacing** around case statement boundaries
- **Multi-line action formatting** with appropriate indentation

### Action Organization
Group related case actions logically, use consistent formatting for single-line versus multi-line actions, and maintain clear visual boundaries between different branches.

### Readability Optimization
Format case statements to support quick visual parsing of decision logic. Use whitespace effectively to create clear visual separation between different execution paths.

## Anti-Patterns

### Avoid These Approaches
- **Inconsistent case label alignment** making structure unclear
- **Poor action block indentation** obscuring code hierarchy
- **Cramped formatting** without adequate spacing
- **Mixed formatting styles** within single case statement
- **Unclear action boundaries** between different cases

Poor case formatting makes decision logic harder to follow and increases the likelihood of errors during code maintenance and enhancement.

*Implementation examples: see samples/al-case-action-formatting.md*
*Related patterns: al-keyword-indentation-rules.md, al-separate-if-else-formatting.md*