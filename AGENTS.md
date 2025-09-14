# AGENTS.md - Repository Context for AI Assistants

## Repository Purpose
This is the **knowledge content repository** for the Business Central Knowledge Base (BCKB) system.

## What This Repo Contains
- **BC Domain Knowledge**: 87+ atomic topics across 24 domains (performance, API design, security, etc.)
- **Methodology Frameworks**: Step-by-step AI workflow guidance for systematic optimization
- **Specialists Definitions**: AI persona configurations with domain expertise and behavioral constraints
- **Strategic Prompts**: High-level prompts for common BC development scenarios
- **Pure Markdown Content**: No code whatsoever - only knowledge content with YAML frontmatter

## What This Repo Does NOT Contain
- TypeScript/JavaScript code (that's in bc-knowledgebase-mcp repository)
- MCP server implementation or protocol handling
- Build processes, dependencies, or package.json files
- Compiled or generated content

## Repository Structure
```
domains/                    # BC knowledge organized by domain
├── performance/           # Performance optimization patterns
├── api-design/           # REST/OData API best practices
├── data-architecture/    # Table design and relationships
├── security/             # Authentication and authorization
└── [20 other domains]/

methodologies/            # AI workflow frameworks
├── phases/              # Systematic analysis phases
└── workflows/           # Complete optimization workflows

specialists/             # AI persona definitions
├── bc-performance-analyst.md
├── bc-security-auditor.md
└── [other specialists]/

prompts/                # Strategic prompts
├── analyze-performance.md
├── review-architecture.md
└── [other prompts]/
```

## Content Standards
- **Atomic Topics**: One focused concept per markdown file
- **YAML Frontmatter**: Rich metadata for AI consumption including:
  - BC version compatibility (`bc_versions: "14+"`)
  - Difficulty level, tags, domain classification
  - Cross-references to related topics
- **Version Awareness**: All content specifies BC version compatibility
- **Consistent Structure**: Follow established patterns for headings, examples, cross-references

## AI Assistant Guidelines When Working With This Repo
1. **Content Focus**: Maintain accuracy of BC technical information
2. **Structure Integrity**: Preserve YAML frontmatter completeness and consistency
3. **Version Compatibility**: Always specify BC version ranges for patterns/features
4. **Cross-References**: Maintain accurate links between related topics
5. **Atomic Principle**: Keep each file focused on one specific concept
6. **NO CODE**: This repository contains NO executable code - knowledge only
7. **Link Integrity**: Ensure internal topic references remain valid

## Common Tasks
- Adding new BC knowledge topics with proper frontmatter
- Updating existing topics for new BC versions
- Creating cross-references between related topics
- Validating YAML frontmatter structure
- Organizing content within domain hierarchies

## Integration
This repository is consumed by the MCP server via git submodule as the embedded knowledge layer (Layer 0).