# AGENTS.md - Repository Context for AI Assistants

## Repository Purpose
This is the **knowledge content repository** for the BC Code Intelligence (bc-code-intel) system.

## What This Repo Contains
- **BC Domain Knowledge**: 87+ atomic topics across 24 domains (performance, API design, security, etc.)
- **Methodology Frameworks**: Step-by-step AI workflow guidance for systematic optimization
- **Specialists Definitions**: 14 AI persona configurations with domain expertise and behavioral constraints
- **Strategic Prompts**: High-level prompts for common BC development scenarios
- **Pure Markdown Content**: No code whatsoever - only knowledge content with YAML frontmatter

## What This Repo Does NOT Contain
- TypeScript/JavaScript code (that's in bc-code-intelligence-mcp repository)
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
├── alex-architect.md        # Strategic Planning & Architecture
├── casey-copilot.md         # AI-Enhanced Development Coach
├── dean-debug.md            # Performance & Troubleshooting
├── eva-errors.md            # Error Handling & Exception Management
├── jordan-bridge.md         # Systems Connection & Event-Driven Design
├── logan-legacy.md          # Legacy Code & Version Migration
├── maya-mentor.md           # Learning & Skill Development
├── morgan-market.md         # AppSource & ISV Business Expert
├── quinn-tester.md          # Quality Validation & Test Design
├── roger-reviewer.md        # Code Quality & Review Standards
├── sam-coder.md             # Efficient Implementation Expert
├── seth-security.md         # Security & Permission Management
├── taylor-docs.md           # Documentation & Knowledge Management
└── uma-ux.md                # User Experience & Interface Design

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

## GitHub Copilot Coding Agent Guidelines

### Content Creation Patterns
When creating new knowledge topics, ALWAYS use this YAML frontmatter template:
```yaml
---
title: "Descriptive Topic Title"
domain: "performance|api-design|security|etc"
bc_versions: "14+|18+|19+|specific-range"
difficulty: "beginner|intermediate|advanced"
tags: ["tag1", "tag2", "tag3"]
related_topics:
  - "path/to/related/topic.md"
  - "another/related/topic.md"
applies_to:
  - "AL Language"
  - "Business Central Server"
  - "Web Client"
last_updated: "2025-01-01"
---
```

### File Naming Conventions
- Use kebab-case for all filenames: `api-pagination-patterns.md`
- Include domain prefix when needed: `performance-caching-strategies.md`
- Keep names descriptive but concise (max 50 characters)
- Avoid abbreviations unless universally understood

### Content Structure Requirements
1. **H1 Title**: Must match YAML frontmatter title exactly
2. **Overview Section**: 2-3 sentence summary of the concept
3. **Implementation Details**: Step-by-step guidance with BC code examples
4. **Best Practices**: Bulleted list of key recommendations
5. **Common Pitfalls**: What to avoid with explanations
6. **Version Notes**: BC version-specific considerations
7. **See Also**: Cross-references to related topics

### Cross-Reference Validation
- Use relative paths: `../performance/caching-strategies.md`
- Verify target files exist before creating references
- Update both directions when creating new relationships
- Use descriptive link text: `[API Pagination Patterns](../api-design/pagination-patterns.md)`

### Domain Organization Rules
```
domains/
├── performance/          # Speed, memory, resource optimization
├── api-design/          # REST, OData, integration patterns
├── security/            # Authentication, authorization, data protection
├── data-architecture/   # Table design, relationships, normalization
├── error-handling/      # Exception management, validation
├── testing/            # Unit, integration, performance testing
└── [other domains]/    # Maintain alphabetical order
```

### Validation Checklist
Before committing new content, verify:
- [ ] YAML frontmatter is complete and valid
- [ ] BC version compatibility is specified
- [ ] Cross-references use relative paths and exist
- [ ] Content is atomic (one focused concept)
- [ ] No executable code is included
- [ ] Examples use realistic BC scenarios
- [ ] Tags are consistent with existing taxonomy

### Common Anti-Patterns to Avoid
- ❌ **Code Repositories**: Never include .cs, .al, .js, or executable files
- ❌ **Generic Examples**: Avoid "MyTable" - use realistic BC entities
- ❌ **Version Ambiguity**: Never omit BC version compatibility
- ❌ **Broken Cross-References**: Always validate links before committing
- ❌ **Topic Drift**: Keep each file focused on ONE concept
- ❌ **Outdated Information**: Update last_updated when making changes

## Common Tasks
- Adding new BC knowledge topics with proper frontmatter and validation
- Updating existing topics for new BC versions with migration notes
- Creating bidirectional cross-references between related topics
- Validating YAML frontmatter structure and consistency
- Organizing content within domain hierarchies and maintaining taxonomy
- Performing content audits for accuracy and BC version compatibility

## GitHub Copilot Integration Workflows

### New Topic Creation Workflow
1. **Domain Classification**: Determine primary domain from existing taxonomy
2. **Version Research**: Identify minimum BC version and any version-specific notes
3. **Template Application**: Use YAML frontmatter template with all required fields
4. **Content Development**: Follow structure requirements for consistency
5. **Cross-Reference Integration**: Add to related topics and update existing references
6. **Validation**: Run through validation checklist before committing

### Content Update Workflow
1. **Impact Assessment**: Identify which BC versions are affected
2. **Cross-Reference Review**: Check if related topics need updates
3. **Version Compatibility**: Update bc_versions field if needed
4. **Last Updated**: Always update timestamp when making changes
5. **Validation**: Ensure all links and references remain valid

### Quality Assurance Patterns
- **Atomic Content Principle**: Each file covers exactly one BC concept
- **Consistent Terminology**: Use official Microsoft BC terminology
- **Practical Examples**: Include realistic Business Central scenarios
- **Version Awareness**: Every pattern specifies BC version compatibility
- **Link Validation**: Cross-references must be bidirectional and accurate

## Integration
This repository is consumed by the MCP server via git submodule as the embedded knowledge layer (Layer 0).