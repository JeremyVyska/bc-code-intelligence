---
title: "Creating Custom BC Knowledge Content"
domain: "chris-config"
difficulty: "intermediate"
bc_versions: "14+"
tags: ["knowledge-creation", "content-authoring", "yaml-frontmatter", "markdown", "overrides"]
related_topics: ["multi-team-layer-configuration", "content-quality-standards"]
applies_to: ["Knowledge Base", "Team Documentation", "Project Customization"]
last_updated: "2025-01-01"
---

# Creating Custom BC Knowledge Content

## Overview

Learn to create high-quality BC knowledge content that perfectly fits your organization's needs. This guide covers writing, structuring, and maintaining knowledge overrides that enhance the embedded knowledge base through the layer system.

**Key Benefit**: Custom knowledge content allows teams to capture specialized patterns, company standards, and project-specific requirements while maintaining compatibility with the broader BC knowledge ecosystem.

## Knowledge Topic Structure

Every knowledge topic follows a consistent structure with YAML frontmatter and organized markdown content:

```yaml
---
# Required Metadata
title: "Descriptive Topic Title"
domain: "performance|api-design|security|etc"
bc_versions: "20+"
difficulty: "beginner|intermediate|advanced|expert"
tags: ["tag1", "tag2", "tag3"]

# Override Configuration (if applicable)
overrides: ["embedded-topic-id"]
merge_strategy: "replace|overlay|smart|append"

# Organization Context
organization_level: "company|team|project"
author: "your.email@company.com"
created: "2025-01-15"
last_modified: "2025-01-20"

# Cross-References
related_topics: ["topic-id-1", "topic-id-2"]
prerequisites: ["prerequisite-topic-id"]
---

# Topic Title

Brief introduction explaining what this topic covers and why it matters.

## Overview
Clear explanation of the concept, pattern, or practice including:
- What it is and when to use it
- Why it's important
- Key benefits for BC development

## Business Central Context
BC-specific considerations:
- Platform integration points
- Version compatibility notes
- Related BC features or modules

## Implementation
Step-by-step guidance with clear instructions:

### Prerequisites
- Required setup or dependencies
- Assumed knowledge or skills

### Step-by-Step Guide
1. **Step 1**: Detailed instruction
   ```al
   // Clear, compilable AL code example
   table 50100 ExampleTable
   {
       DataClassification = CustomerContent;
       
       fields
       {
           field(1; "Entry No."; Integer)
           {
               AutoIncrement = true;
           }
       }
   }
   ```

## Best Practices
Proven recommendations:
- ✅ Use clear, descriptive naming conventions
- ✅ Implement proper error handling
- ✅ Follow BC coding standards
- ❌ Avoid hard-coded values
- ❌ Don't skip validation

## Version Considerations
BC version compatibility and migration notes

## Troubleshooting
Common issues and solutions

## Related Topics
Links to related knowledge content
```

## YAML Frontmatter Reference

### Required Fields
```yaml
---
title: "Descriptive Topic Title"        # Clear, specific title
domain: "performance"                   # Domain classification
bc_versions: "20+"                     # Version compatibility
difficulty: "intermediate"             # Complexity level
tags: ["tag1", "tag2"]                # Minimum 2 descriptive tags
---
```

### Domain Classifications
Choose the most appropriate domain for your content:

```yaml
# Technical domains
domain: "performance"        # Optimization, SIFT, query tuning
domain: "api-design"        # REST, OData, authentication
domain: "security"          # Access control, data protection
domain: "architecture"      # System design, patterns
domain: "data-architecture" # Tables, relationships
domain: "debugging"         # Troubleshooting, diagnostics

# Process domains
domain: "best-practices"    # General guidelines
domain: "error-handling"    # Exception management
domain: "testing"          # Quality assurance
domain: "deployment"       # Release management

# Specialist domains
domain: "alex-architect"    # Architecture and design
domain: "dean-debug"       # Performance and debugging
domain: "sam-coder"        # Implementation patterns
domain: "chris-config"     # Configuration and customization
```

### Difficulty Level Guidelines
```yaml
difficulty: "beginner"      # Basic concepts, getting started
# - Assumes minimal BC knowledge
# - Step-by-step instructions
# - Explains basic concepts

difficulty: "intermediate"  # Practical implementation
# - Assumes basic BC development knowledge
# - Real-world scenarios
# - Best practices focus

difficulty: "advanced"      # Complex patterns, optimization
# - Assumes solid BC experience
# - Performance considerations
# - Integration challenges

difficulty: "expert"        # Cutting-edge, specialized
# - Assumes deep BC expertise
# - Advanced patterns
# - Platform limitations
```

### Effective Tagging Strategy
```yaml
# Performance-related tags
tags: ["sift", "optimization", "query-performance", "memory-management"]

# API and integration tags  
tags: ["rest", "odata", "authentication", "integration", "webhooks"]

# Architecture and design tags
tags: ["design-patterns", "microservices", "event-driven", "scalability"]

# Development process tags
tags: ["testing", "debugging", "devops", "code-review"]

# Business domain tags
tags: ["manufacturing", "retail", "financial", "healthcare"]
```

## Override Configuration

### Simple Content Override
Replace embedded content completely:
```yaml
---
title: "Company SIFT Optimization Standards"
domain: "performance"
overrides: ["sift-technology-fundamentals"]
merge_strategy: "replace"
organization_level: "company"
---
```

### Content Enhancement
Add to existing embedded content:
```yaml
---
title: "Team-Specific API Authentication Patterns"
domain: "api-design"
overrides: ["api-authentication-patterns"]
merge_strategy: "append"
organization_level: "team"
sections_to_enhance: ["implementation", "examples"]
---
```

### Conditional Overrides
Apply overrides based on conditions:
```yaml
---
title: "Production Environment Security Standards"
domain: "security"
override_conditions:
  environment: ["production"]
  bc_version: "23+"
---
```

## Code Example Standards

### AL Code Examples
```al
// Always include context comments
table 50100 "Customer Analytics"
{
    // Purpose: Store customer performance metrics
    DataClassification = CustomerContent;
    
    fields
    {
        field(1; "Customer No."; Code[20])
        {
            // Links to standard Customer table
            TableRelation = Customer."No.";
            Caption = 'Customer No.';
        }
        
        field(10; "Total Sales Amount"; Decimal)
        {
            // Calculated via SIFT for performance
            FieldClass = FlowField;
            CalcFormula = Sum("Sales Line".Amount WHERE("Sell-to Customer No." = FIELD("Customer No.")));
            Caption = 'Total Sales Amount';
        }
    }
    
    keys
    {
        key(PK; "Customer No.")
        {
            Clustered = true;
        }
        
        // SIFT key for aggregation performance
        key(SalesAnalysis; "Customer No.")
        {
            SumIndexFields = "Total Sales Amount";
        }
    }
}
```

### Configuration Examples
```json
// JSON configuration with explanatory comments
{
  "performance_settings": {
    "enable_sift": true,           // Enables SIFT optimization
    "max_records": 10000,         // Prevents memory issues
    "timeout_seconds": 30         // Query timeout protection
  },
  "cache_settings": {
    "enabled": true,
    "ttl_minutes": 15            // Cache expiration time
  }
}
```

## Content Development Workflow

### 1. Planning Phase
```markdown
## Topic Planning Checklist
- [ ] Identify knowledge gap or override need
- [ ] Define target audience and use case
- [ ] Research existing related topics
- [ ] Plan topic scope and depth
- [ ] Choose appropriate domain and tags
- [ ] Determine BC version compatibility
```

### 2. Content Creation Phase
```markdown
## Content Creation Checklist
- [ ] Write compelling title and overview
- [ ] Create YAML frontmatter with required fields
- [ ] Develop step-by-step implementation guide
- [ ] Include practical, compilable code examples
- [ ] Add best practices and anti-patterns
- [ ] Write troubleshooting section
- [ ] Add cross-references to related topics
```

### 3. Quality Assurance Phase
```markdown
## Quality Review Checklist
- [ ] YAML frontmatter syntax is valid
- [ ] Markdown renders correctly
- [ ] Code examples compile successfully
- [ ] Cross-references resolve correctly
- [ ] Content accuracy verified
- [ ] Writing clarity and flow reviewed
- [ ] Version compatibility confirmed
```

## Layer Integration

### Project-Level Overrides
Create content in your project's override directory:
```
bc-code-intel-overrides/
├── domains/
│   ├── performance/
│   │   └── our-sift-standards.md
│   ├── security/
│   │   └── company-auth-patterns.md
│   └── chris-config/
│       └── project-specific-config.md
└── specialists/
    └── company-reviewer.md
```

### Team Git Repository
Structure team knowledge in a Git repository:
```
team-bc-knowledge/
├── domains/
│   ├── performance/
│   │   ├── team-optimization-patterns.md
│   │   └── performance-testing-standards.md
│   ├── api-design/
│   │   └── team-integration-patterns.md
│   └── shared/
│       └── team-coding-conventions.md
├── specialists/
│   └── team-specialist-config.md
└── methodologies/
    └── team-code-review-process.md
```

### Company Standards Repository
Maintain organization-wide standards:
```
company-bc-standards/
├── domains/
│   ├── security/
│   │   ├── corporate-security-standards.md
│   │   └── data-classification-requirements.md
│   ├── architecture/
│   │   └── enterprise-integration-patterns.md
│   └── best-practices/
│       └── corporate-coding-standards.md
└── policies/
    ├── approval-requirements.md
    └── review-processes.md
```

## Quality Standards

### Content Quality Metrics
- **Completeness**: All required sections present
- **Clarity**: Clear, actionable instructions
- **Examples**: Working, realistic code samples
- **Currency**: Up-to-date with current BC versions
- **Cross-references**: Proper links to related content

### Maintenance Standards
```yaml
# Quality gates for topic updates
quality_standards:
  minimum_completeness: 80
  required_sections: ["overview", "implementation", "best-practices"]
  code_example_requirement: true
  cross_reference_minimum: 3
  version_compatibility_current: true
```

## Best Practices

### Content Writing
- **Be specific**: Focus on actionable guidance
- **Use examples**: Include working AL code samples
- **Link appropriately**: Reference related topics
- **Update regularly**: Keep content current with BC versions
- **Test thoroughly**: Verify all code examples compile

### Organization
- **Consistent structure**: Follow established patterns
- **Clear taxonomy**: Use appropriate domains and tags
- **Logical progression**: Order content from basic to advanced
- **Proper versioning**: Clearly indicate BC compatibility

### Collaboration
- **Document decisions**: Explain why overrides exist
- **Review processes**: Establish content quality gates
- **Version control**: Track changes and contributors
- **Training**: Help team members contribute effectively

## Common Anti-Patterns

### Content Issues
- ❌ **Vague titles**: "Performance Tips" instead of specific guidance
- ❌ **Missing context**: Code without explanation
- ❌ **Outdated examples**: Using deprecated BC features
- ❌ **Broken references**: Links to non-existent topics

### Organizational Issues
- ❌ **Too many overrides**: Overriding everything instead of selective enhancement
- ❌ **Conflicting content**: Multiple teams providing contradictory guidance
- ❌ **No maintenance**: Creating content without ongoing updates
- ❌ **Poor governance**: No review or approval processes

## Related Topics

- **[Multi-Team Layer Configuration](multi-team-layer-configuration)**: Setting up organizational knowledge layers
- **[Content Quality Standards](content-quality-standards)**: Maintaining high-quality knowledge
- **[Specialist Customization](specialist-customization)**: Creating custom AI specialists

---

*Master content creation to build a comprehensive BC knowledge base that serves your organization's specific needs while maintaining compatibility with the broader ecosystem.*