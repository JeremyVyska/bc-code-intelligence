---
title: "AL Lonely Repeat Pattern"
domain: "sam-coder"
difficulty: "intermediate"
bc_versions: "14+"
tags: ["repeat-loops", "control-flow", "code-patterns"]
samples: "samples/al-lonely-repeat-pattern.md"
type: "improvement-pattern"
category: "language-fundamentals"
pattern_type: "good"
severity: "medium"
impact_level: "medium"
improvement_suggestion: "Avoid lonely repeat statements by ensuring proper until conditions and loop structure"
implementation_steps: ["identify-repeat-patterns", "validate-until-conditions", "ensure-proper-structure"]
validation_criteria: ["complete-repeat-until-pairs", "proper-loop-structure"]
technical_areas: ["al-code"]
---

# AL Lonely Repeat Pattern

## Overview

The lonely repeat pattern in AL occurs when repeat statements lack proper until conditions or have incomplete loop structures. Understanding proper repeat-until loop construction prevents infinite loops and ensures reliable iteration patterns in Business Central code.

**Key Principle**: Every repeat statement must have a corresponding until condition that provides a reliable loop exit strategy.

## Strategic Framework

### Loop Structure Integrity
- **Complete Pairs**: Every repeat must have matching until condition
- **Exit Conditions**: Reliable termination logic for all loops
- **Logic Validation**: Ensure until conditions can actually be met
- **Infinite Loop Prevention**: Proper loop variable modification patterns

### Control Flow Reliability
Repeat-until loops must have well-defined termination conditions that guarantee loop completion under normal execution scenarios.

## Architecture Focus

### Repeat-Until Patterns
- **Condition Logic**: Until conditions that provide reliable loop exit
- **Variable Modification**: Loop variables that progress toward termination
- **Error Handling**: Safeguards against infinite loop scenarios
- **Performance Considerations**: Efficient loop structures for data processing

### Loop Design Principles
Structure repeat-until loops with clear progression logic and reliable termination conditions. Avoid patterns that could lead to infinite execution or unpredictable behavior.

## Best Practices

### Proper Loop Construction
- **Clear until conditions** that can be reliably evaluated
- **Loop variable progression** ensuring movement toward termination
- **Boundary condition handling** for edge cases
- **Error prevention** through validation and safeguards
- **Performance optimization** for large data set processing

### Termination Logic
Design until conditions that provide multiple exit paths when appropriate and include safeguards against infinite loop scenarios in data processing operations.

### Code Reliability
Structure repeat loops with clear termination logic and appropriate error handling to ensure reliable execution in all Business Central scenarios.

## Anti-Patterns

### Avoid These Approaches
- **Repeat without until** creating incomplete loop structures
- **Unreachable until conditions** that cannot be satisfied
- **No loop variable progression** preventing termination
- **Complex until logic** that obscures termination conditions
- **Missing error handling** for loop failure scenarios

Lonely repeat patterns create unreliable code that may cause infinite loops or unpredictable behavior in Business Central applications.

*Implementation examples: see samples/al-lonely-repeat-pattern.md*
*Related patterns: al-statement-per-line-rule.md, al-readability-optimization.md*