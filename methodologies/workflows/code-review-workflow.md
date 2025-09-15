---
type: workflow
category: quality-assurance
name: bc-code-review-workflow
workflow_type: checklist
phases:
  - name: preparation
    title: "Pre-Review Preparation"
    required: true
    tasks:
      - "Ensure all tests pass"
      - "Run static analysis tools"
      - "Check code formatting compliance"
      - "Verify documentation is updated"
  - name: performance-review
    title: "Performance Analysis"
    required: true
    tasks:
      - "Check for SetLoadFields usage"
      - "Verify efficient query patterns"
      - "Validate SIFT usage for summations"
      - "Review loop efficiency"
      - "Check for unnecessary CalcFields calls"
  - name: security-review
    title: "Security & Validation"
    required: true
    tasks:
      - "Validate user input handling"
      - "Check permission requirements"
      - "Review error handling"
      - "Verify data validation"
  - name: maintainability
    title: "Code Maintainability"
    required: false
    tasks:
      - "Check naming conventions"
      - "Review code complexity"
      - "Validate commenting"
      - "Assess testability"
related_topics:
  - code-quality-standards
  - performance-review-checklist
  - security-best-practices
domains: [quality-assurance, performance, security]
difficulty: intermediate
estimated_duration: "30-45 minutes"
prerequisites: ["Basic AL knowledge", "Understanding of BC architecture"]
tags: [workflow, code-review, quality-assurance, checklist]
---

# Business Central Code Review Workflow

## Overview
A systematic approach to reviewing AL code for Business Central extensions, focusing on performance, security, and maintainability.

## Workflow Phases

### 1. Pre-Review Preparation (Required)
Before starting the review, ensure:
- [ ] All automated tests pass
- [ ] Static analysis tools show no critical issues
- [ ] Code follows formatting standards
- [ ] Documentation reflects changes

### 2. Performance Analysis (Required)
Critical performance checks:
- [ ] **SetLoadFields**: Used before FindSet() when only specific fields needed
- [ ] **Query Patterns**: Efficient use of SetRange, SetFilter, SetCurrentKey
- [ ] **SIFT Usage**: CalcSums instead of manual summation loops
- [ ] **Loop Efficiency**: Minimal processing inside repeat-until loops
- [ ] **CalcFields**: Batched rather than individual calls

### 3. Security & Validation (Required)
Security-focused review:
- [ ] **Input Validation**: All user inputs properly validated
- [ ] **Permissions**: Appropriate permission sets required
- [ ] **Error Handling**: Graceful error handling without data exposure
- [ ] **Data Validation**: Business rules properly enforced

### 4. Code Maintainability (Optional)
Quality improvements:
- [ ] **Naming**: Clear, consistent naming conventions
- [ ] **Complexity**: Functions under 50 lines, minimal nesting
- [ ] **Comments**: Complex logic explained
- [ ] **Testability**: Code structure supports unit testing

## Review Checklist Tools
Use these questions for each phase:

**Performance**: "Will this code perform well with 10,000+ records?"
**Security**: "What happens if a malicious user provides unexpected input?"
**Maintainability**: "Can another developer understand this in 6 months?"

## Common Issues to Flag
- Manual summation loops (use CalcSums)
- Missing SetLoadFields on large datasets
- Hardcoded values that should be configurable
- Missing error handling on external calls
- Complex nested logic without comments