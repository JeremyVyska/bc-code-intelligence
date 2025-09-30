---
title: "Multi-Team Layer Configuration for Organizations"
domain: "chris-config"
difficulty: "intermediate"
bc_versions: "14+"
tags: ["configuration", "enterprise", "layers", "git-repositories", "team-collaboration"]
related_topics: ["knowledge-overrides", "layer-system-architecture"]
applies_to: ["MCP Server", "VS Code Integration", "Team Environments"]
last_updated: "2025-01-01"
---

# Multi-Team Layer Configuration for Organizations

## Overview

Configure the BC Code Intelligence MCP Server for organizations needing shared team knowledge, centralized BC standards, and Git-based knowledge management. This guide covers the simple but effective layer-based override system for organizational hierarchy.

**Key Benefit**: Teams can share BC knowledge while maintaining project-specific customizations through a priority-based layer system.

## What This System Actually Provides

### ✅ Real Capabilities
- **Local file overrides** - Project-specific knowledge in `./bc-code-intel-overrides/`
- **Git repository layers** - Team/company knowledge from Git repos  
- **Priority-based resolution** - Higher priority layers override lower ones
- **Basic authentication** - Token-based auth for private Git repos
- **Embedded knowledge base** - 87+ BC topics across 24 domains
- **Simple configuration** - JSON/YAML config files

### ❌ What This Does NOT Do
- Multi-tenant architecture (doesn't exist)
- Azure AD integration (not implemented)
- Business unit isolation (fantasy)
- Enterprise monitoring dashboards (not built)
- Kubernetes deployments (unnecessary complexity)

## Layer Architecture

The system uses a straightforward override hierarchy:

```
Project Layer (Priority 100)     ← ./bc-code-intel-overrides/
      ↓ overrides
Team Layer (Priority 50)         ← Git repo: team-bc-knowledge
      ↓ overrides  
Company Layer (Priority 25)      ← Git repo: company-bc-standards
      ↓ overrides
Embedded Layer (Priority 0)      ← Built-in BC knowledge base
```

## VS Code MCP Integration Setup

Most users integrate through VS Code's `mcp.json` configuration:

### Basic VS Code Setup
```json
{
  "mcpServers": {
    "bc-code-intelligence": {
      "command": "npx",
      "args": ["bc-code-intelligence-mcp"],
      "env": {
        "GITHUB_TOKEN": "your_github_token_here"
      }
    }
  }
}
```

### Project-Specific Configuration
```bash
# In your BC project root
echo '{
  "layers": [
    {"name": "embedded", "priority": 0, "source": {"type": "embedded"}, "enabled": true},
    {"name": "company", "priority": 25, "source": {"type": "git", "url": "https://github.com/yourcompany/bc-standards"}, "enabled": true, "auth": {"type": "token", "token_env_var": "GITHUB_TOKEN"}},
    {"name": "project", "priority": 100, "source": {"type": "local", "path": "./bc-code-intel-overrides"}, "enabled": true}
  ]
}' > bc-code-intel-config.json
```

### User-Wide Configuration
```bash
# Windows (PowerShell/CMD)
echo '{...}' > %USERPROFILE%\bc-code-intel-config.json

# macOS/Linux
echo '{...}' > ~/bc-code-intel-config.json
```

## Multi-Team Configuration Example

```json
{
  "layers": [
    {
      "name": "embedded",
      "priority": 0,
      "source": { "type": "embedded" },
      "enabled": true
    },
    {
      "name": "company-standards",
      "priority": 25,
      "source": {
        "type": "git",
        "url": "https://github.com/yourcompany/bc-standards",
        "branch": "main"
      },
      "enabled": true,
      "auth": {
        "type": "token",
        "token_env_var": "GITHUB_TOKEN"
      }
    },
    {
      "name": "team-practices",
      "priority": 50,
      "source": {
        "type": "git", 
        "url": "https://github.com/yourcompany/team-bc-knowledge",
        "branch": "main"
      },
      "enabled": true,
      "auth": {
        "type": "token",
        "token_env_var": "GITHUB_TOKEN"
      }
    },
    {
      "name": "project-overrides",
      "priority": 100,
      "source": {
        "type": "local",
        "path": "./bc-code-intel-overrides"
      },
      "enabled": true
    }
  ],
  "cache": {
    "strategy": "moderate",
    "ttl": {
      "git": "1h",
      "local": "immediate",
      "embedded": "permanent"
    }
  }
}
```

## Layer Types and Use Cases

### Embedded Layer (Priority 0)
- **Content**: 87+ topics across 24 domains, 14 specialist definitions
- **Use case**: Foundation knowledge everyone gets
- **Always available**: Provides fallback when other layers fail

### Company Standards Layer (Priority 25)
- **Content**: Organization BC coding standards and approved patterns
- **Use case**: Organization-wide standards that override defaults
- **Authentication**: Usually requires private Git repository access

### Team Practice Layer (Priority 50)
- **Content**: Team-specific approaches and specialized knowledge
- **Use case**: Team expertise that extends company standards
- **Collaboration**: Multiple teams can have different practice repositories

### Project Override Layer (Priority 100)
- **Content**: Project-specific customizations and experiments
- **Use case**: Project needs that override everything else
- **Local files**: No authentication required, immediate changes

## Configuration File Discovery

The MCP server searches for configuration files in this order:
1. `./bc-code-intel-config.json` (current directory)
2. `./bc-code-intel-config.yaml` (current directory)
3. **Windows**: `%USERPROFILE%\bc-code-intel-config.json`
4. **Windows**: `%USERPROFILE%\bc-code-intel-config.yaml`
5. **macOS/Linux**: `~/bc-code-intel-config.json`
6. **macOS/Linux**: `~/bc-code-intel-config.yaml`

## Authentication Options

### GitHub Token Authentication
```bash
# Set environment variable
export GITHUB_TOKEN="ghp_your_token_here"
```

```json
{
  "auth": {
    "type": "token",
    "token_env_var": "GITHUB_TOKEN"
  }
}
```

### SSH Key Authentication
```json
{
  "auth": {
    "type": "ssh",
    "key_path": "~/.ssh/id_rsa"
  }
}
```

## Setup Checklist

### Basic Multi-Team Setup
- [ ] Create company BC standards repository  
- [ ] Define team knowledge repository structure
- [ ] Set up GitHub token for private repositories
- [ ] Create configuration file with layer priorities
- [ ] Test layer resolution with sample content
- [ ] Train team leads on knowledge contribution

### Content Creation
- [ ] Migrate existing BC standards to markdown format
- [ ] Create team-specific specialist definitions (optional)
- [ ] Document local coding patterns and decisions
- [ ] Set up project override directory structure
- [ ] Create contribution guidelines for team knowledge

## Common Issues and Solutions

### Layer Not Loading
```bash
# Debug layer loading
bc-code-intel --debug-layers
bc-code-intel --test-layer company-standards
```

**Common causes:**
- Invalid Git URL or authentication
- Wrong branch name in configuration
- Network connectivity issues
- Invalid JSON/YAML in config file

### Authentication Failed
```bash
# Test authentication
export GITHUB_TOKEN="your_token"
bc-code-intel --test-auth
```

**Common causes:**
- Expired or invalid GitHub token
- Token doesn't have repository access
- SSH key not configured properly
- Repository is private but no auth configured

## ROI and Cost-Benefit Analysis

### Implementation Costs
- **Software**: Free (open source)
- **Setup time**: 2-4 hours for basic multi-team setup
- **Maintenance**: Minimal - just updating Git repositories
- **Training**: 30 minutes per developer

### Expected Benefits
- **Consistent practices**: Teams follow the same BC patterns
- **Knowledge sharing**: Easy to share team discoveries
- **Faster onboarding**: New developers see team-specific guidance
- **Flexible customization**: Projects can override anything when needed

### Team Size ROI
- **Small teams (2-5 devs)**: Pays for itself in first month
- **Medium teams (6-15 devs)**: 3-5x productivity improvement on BC questions
- **Large teams (15+ devs)**: Significant standardization and knowledge leverage

## Best Practices

### Security Considerations
- **Validate sources**: Only load from trusted repositories
- **Use authentication**: Secure access to private knowledge
- **Limit permissions**: Use read-only tokens when possible

### Configuration Management
- **Start simple**: Begin with embedded + project layers
- **Add gradually**: Add organizational layers as needed
- **Document overrides**: Clearly explain why overrides exist
- **Version control**: Keep layer configurations in git
- **Test changes**: Validate layer configurations before sharing

### Anti-Patterns to Avoid
- ❌ **Too many layers**: More than 4-5 layers becomes complex
- ❌ **Conflicting priorities**: Multiple layers with same priority
- ❌ **No fallback**: Always ensure embedded layer is available
- ❌ **Complex authentication**: Keep auth simple and documented

## Related Topics

- **[Knowledge Content Creation](knowledge-overrides)**: How to create custom BC knowledge content
- **[Layer System Architecture](layer-system-architecture)**: Understanding the technical implementation
- **[Specialist Customization](specialist-customization)**: Creating custom AI specialists

---

*This configuration guide enables organizations to scale BC knowledge sharing while maintaining flexibility for team and project needs.*