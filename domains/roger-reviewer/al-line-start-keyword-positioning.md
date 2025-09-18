---
title: "AL Line Start Keyword Positioning"
domain: "roger-reviewer"
difficulty: "intermediate"
bc_versions: "14+"
tags: ["keywords", "line-positioning", "code-structure"]
samples: "samples/al-line-start-keyword-positioning.md"
type: "improvement-pattern"
category: "code-formatting"
pattern_type: "good"
severity: "medium"
impact_level: "medium"
improvement_suggestion: "Position AL keywords at line start with consistent alignment to show code structure"
implementation_steps: ["identify-keyword-patterns", "apply-positioning-rules", "validate-alignment"]
validation_criteria: ["consistent-keyword-positioning", "clear-structure-indication"]
technical_areas: ["al-code"]
---

# AL Line Start Keyword Positioning

## Overview

Strategic positioning of AL keywords at line starts creates clear visual anchors that help developers quickly identify code structure, control flow patterns, and logical organization. Consistent keyword positioning improves code scanability and reduces cognitive load during code review.

**Key Principle**: Position keywords at line starts with consistent alignment to create clear visual structure markers.

## Strategic Framework

### Visual Structure Creation
- **Keyword Anchoring**: Keywords as visual reference points for code organization
- **Scan Optimization**: Positioning that supports quick visual code parsing
- **Structure Indication**: Keyword placement that reveals code hierarchy
- **Consistency Standards**: Uniform positioning patterns across codebase

### Readability Enhancement
Line start keyword positioning helps developers quickly identify major code sections, control flow changes, and logical boundaries within AL procedures and functions.

## Architecture Focus

### Positioning Patterns
- **Control Flow Keywords**: if, else, while positioned as structure markers
- **Block Delimiters**: begin, end placement showing scope boundaries
- **Declaration Keywords**: var, procedure positioning for clear identification
- **Statement Keywords**: Consistent placement for readability optimization

### Alignment Strategies
Establish consistent rules for keyword alignment that work well with AL language structure and support effective code organization patterns.

## Best Practices

### Keyword Placement Rules
- **Line start positioning** for major structural keywords
- **Consistent alignment** within code blocks and sections
- **Clear separation** from surrounding code elements
- **Logical grouping** of related keyword structures
- **Visual hierarchy** through strategic keyword positioning

### Structure Clarity
Use keyword positioning to create clear visual markers for code organization. Position keywords to support quick identification of major code sections and control flow patterns.

### Consistency Application
Apply keyword positioning rules consistently across all AL objects. Establish team standards for keyword placement that enhance code readability and maintainability.

## Anti-Patterns

### Avoid These Approaches
- **Inconsistent keyword alignment** creating visual confusion
- **Mid-line keyword placement** reducing structure visibility
- **Random positioning** without logical organization principles
- **Poor separation** from surrounding code elements
- **Mixed positioning styles** within single AL object

Inconsistent keyword positioning makes code structure harder to parse and reduces the effectiveness of visual code organization techniques.

*Implementation examples: see samples/al-line-start-keyword-positioning.md*
*Related patterns: al-keyword-indentation-rules.md, al-statement-per-line-rule.md*