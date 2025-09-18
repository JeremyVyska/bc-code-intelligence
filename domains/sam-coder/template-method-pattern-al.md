---
title: "Template Method Pattern in AL"
domain: "sam-coder"
difficulty: "intermediate"
bc_versions: "14+"
tags: ["template-method", "behavioral-patterns", "algorithm-structure", "inheritance"]
prerequisites: ["object-orientation-al", "inheritance-patterns", "virtual-methods"]
samples: "samples/template-method-pattern.md"
related_topics: ["business-process-template-patterns", "type-safe-operations-al"]
---

# Template Method Pattern in AL

## Overview

The Template Method Pattern defines the skeleton of an algorithm in a base class, allowing subclasses to override specific steps without changing the algorithm's structure. In AL development, this pattern enables consistent process execution while providing flexibility for customization.

**Key Benefit**: Provides a stable algorithm framework that ensures consistent execution while enabling customization of specific steps through inheritance.

## Core Pattern Architecture

### Algorithm Skeleton Definition
Define the overall algorithm structure in a base codeunit or interface, establishing the sequence of operations and control flow.

### Extension Points
Identify specific steps in the algorithm that can be customized by subclasses while maintaining the overall algorithm integrity.

### Invariant Operations
Implement operations that remain constant across all algorithm implementations, providing stability and consistency.

## Implementation Strategies

### Virtual Method Framework
Use virtual methods in AL to create extension points that subclasses can override while maintaining the template algorithm structure.

### Interface-Based Templates
Design template patterns using interfaces to define algorithm contracts that implementing classes must fulfill.

### Event-Driven Templates
Integrate template methods with AL event systems to provide extension points without requiring inheritance.

## AL-Specific Patterns

### Codeunit Template Hierarchy
Create codeunit hierarchies where base codeunits define template algorithms and derived codeunits customize specific operations.

### Page Template Patterns
Implement page template patterns that provide consistent user interaction flows while allowing customization of specific behaviors.

### Report Template Framework
Design report templates that define consistent report generation algorithms while enabling customization of data processing and formatting.

## Advanced Template Techniques

### Multi-Phase Templates
Design template algorithms that support multiple phases of execution, enabling complex customization scenarios.

### Conditional Template Execution
Implement template methods that can conditionally execute different algorithm branches based on runtime conditions or configuration.

### Nested Template Patterns
Create nested template structures where template methods can invoke other template methods, enabling complex algorithm composition.

## Advanced Implementation

### Process Integration
Design templates for business processes, data operations, and workflow automation with consistent error handling.

### Performance & Validation
Optimize template efficiency, implement lazy evaluation, and provide consistent validation approaches.

## Implementation Guidelines

### Testing & Documentation
Validate template structure, test customizations, and provide comprehensive documentation with clear customization guidelines.

### Framework Integration
Integrate with existing Business Central frameworks and patterns for consistent implementation.

### Extension Integration
Design templates that work effectively with AL extension mechanisms and AppSource publishing requirements.

### Cross-Module Templates
Create template patterns that can work across different modules while maintaining consistent behavior.

## Best Practices

Keep algorithms focused, minimize extension points, document contracts, and implement comprehensive error handling.

## Common Pitfalls

Avoid template complexity and inflexible designs that reduce effectiveness and customization opportunities.

*Complete template method examples: samples/template-method-pattern.md*
*Business process templates: business-process-template-patterns.md*
*Type-safe implementations: type-safe-operations-al.md*