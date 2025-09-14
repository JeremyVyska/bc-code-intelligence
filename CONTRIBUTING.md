# Contributing to Business Central Knowledge Base

## Overview
This repository contains pure knowledge content for Business Central best practices. All content is markdown with YAML frontmatter.

## Content Standards

### File Structure
Each knowledge topic follows this structure:
```markdown
---
title: "Topic Title"
domain: "performance"
difficulty: "intermediate"
bc_versions: "14+"
tags: ["sift", "optimization", "database"]
related_topics: ["topic-id-1", "topic-id-2"]
---

# Topic Title

Content here following established patterns...
```

### Quality Requirements
- **Atomic Focus**: One concept per file
- **BC Version Compatibility**: Always specify version ranges
- **Complete Frontmatter**: All required YAML fields present
- **Cross-References**: Link to related topics where appropriate
- **Consistent Formatting**: Follow existing markdown patterns

## Submission Process
1. Fork the repository
2. Create content following established patterns in appropriate domain directory
3. Ensure YAML frontmatter is complete and valid
4. Test that internal links work correctly
5. Submit pull request with clear description of the knowledge added/updated

## Content Validation
Pull requests are automatically validated for:
- YAML frontmatter completeness
- BC version compatibility format
- Internal link integrity
- Proper domain/directory placement
- Markdown formatting consistency

## Domain Organization
Place content in appropriate domain directories:
- `domains/performance/` - Performance optimization
- `domains/api-design/` - REST/OData patterns
- `domains/security/` - Authentication/authorization
- `domains/data-architecture/` - Table design
- [See full domain list in repository structure]

## Getting Help
- Review existing topics for patterns
- Check AGENTS.md for AI assistant guidance
- Open issues for questions about content organization