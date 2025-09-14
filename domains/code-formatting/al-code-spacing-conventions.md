---
title: "AL Code Spacing and Indentation Conventions"
domain: "code-formatting"
difficulty: "beginner"
bc_versions: "14+"
tags: ["formatting", "indentation", "code-style", "readability"]
prerequisites: ["al-syntax"]
---

# AL Code Spacing and Indentation Conventions

## Overview

Consistent spacing and indentation in AL code improves readability, reduces cognitive load, and enables better collaboration across development teams. Following established conventions ensures code maintainability and professional appearance.

**Foundation Principle**: Use consistent 4-space indentation throughout AL code with strategic white space to enhance logical grouping and flow comprehension.

## Strategic Framework

### Indentation Standards
- **4-space indentation**: Standard for AL code blocks and nested structures
- **No mixed tabs/spaces**: Consistent space-only indentation
- **Logical nesting**: Each nested level increases indentation by 4 spaces
- **Alignment consistency**: Maintain alignment patterns within code sections

### White Space Strategy
- **Logical grouping**: Separate related code blocks with blank lines
- **Operator spacing**: Consistent spacing around operators and assignments
- **Parameter formatting**: Clear spacing in procedure calls and definitions
- **Object boundaries**: Appropriate spacing between object sections

## Architecture Patterns

### Object Structure Spacing
Organize object definitions with consistent spacing between properties, fields, and procedures. Use blank lines to separate logical sections within objects.

### Procedure Formatting
Structure procedure definitions with proper spacing around parameters, local variables, and code blocks. Maintain consistent patterns for procedure calls and nested logic.

### Expression and Statement Spacing
Apply consistent spacing around operators, assignment statements, and conditional expressions to enhance readability and reduce parsing errors.

## Implementation Guidelines

### Standard Patterns
- **Variable declarations**: Group related variables with consistent spacing
- **Control structures**: Proper spacing around if/then/else, for loops, and case statements
- **Object references**: Consistent spacing in field references and procedure calls
- **Multi-line statements**: Logical line breaks with appropriate continuation indentation

### Special Formatting Cases
- **Long parameter lists**: Break parameters across lines with consistent indentation
- **Complex expressions**: Use spacing and line breaks to clarify operator precedence
- **Nested structures**: Maintain clear indentation hierarchy in complex logic
- **String concatenation**: Clear spacing around concatenation operators

## Best Practices

### Consistency Standards
- **Team conventions**: Establish and maintain team-wide formatting standards
- **Automated formatting**: Use AL formatter tools to ensure consistency
- **Code reviews**: Include formatting checks in review processes
- **Documentation**: Maintain formatting guidelines for team reference

### Readability Optimization
- **Logical grouping**: Use white space to show code organization
- **Visual hierarchy**: Indentation reflects logical code structure
- **Scanning efficiency**: Format code for quick visual parsing
- **Error reduction**: Clear formatting reduces misreading and mistakes

### Tool Integration
Configure development environments to automatically apply consistent formatting. Use linting tools to enforce spacing conventions across the codebase.

## Anti-Patterns

### Avoid These Approaches
- **Inconsistent indentation**: Mixed spaces, tabs, or varying indent levels
- **Excessive blank lines**: Too much white space that breaks code flow
- **Cramped formatting**: Insufficient spacing that reduces readability
- **Misaligned parameters**: Inconsistent parameter formatting across procedures
- **Ignored formatting tools**: Manual formatting instead of automated tools

