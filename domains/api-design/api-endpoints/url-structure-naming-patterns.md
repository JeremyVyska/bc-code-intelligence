---
title: "API Page URL Structure and Entity Naming Patterns"
domain: "api-design"
difficulty: "beginner"
bc_versions: "14+"
tags: ["api-endpoints", "url-structure", "naming-conventions"]
samples: "samples/api-url-naming.md"
---

# API Page URL Structure and Entity Naming Patterns

## Overview

Business Central API pages follow predictable URL patterns that directly correlate with entity naming conventions. Understanding these patterns is essential for API design consistency and client integration predictability.

The URL structure determines how external systems discover and interact with your BC data, making proper naming a critical architectural decision that affects long-term maintainability and integration success.

## URL Structure Components

### Base URL Pattern
Business Central APIs follow the standard OData URL structure:
```
https://{server}/BC{version}/api/{publisher}/{group}/{version}/{entity}
```

Each component serves a specific purpose in API discoverability and versioning strategy.

### Entity Naming Requirements
Entity names in API pages must follow specific BC conventions:
- Must be plural nouns (e.g., "customers", not "customer")  
- Use lowercase with hyphens for multi-word entities
- Cannot exceed 30 characters in length
- Must be unique within the API group scope

### Publisher and Group Naming
Publisher identifies the extension owner, while group logically organizes related entities. Both impact the final URL structure and should follow consistent organizational patterns.

## Naming Pattern Guidelines

### Standard Entity Patterns
Follow established BC naming patterns for consistency with Microsoft's own APIs:
- Master data: "customers", "vendors", "items"
- Transactional data: "sales-orders", "purchase-invoices" 
- Setup data: "payment-terms", "shipping-methods"

### Multi-Word Entity Handling
When entity names contain multiple words, use hyphen separation for URL readability while maintaining camelCase in AL object naming for code consistency.

### Abbreviation Strategy
Avoid abbreviations in entity names unless they are widely understood business terms. Full words improve API discoverability and reduce integration confusion.

## Implementation Considerations

### EntityName vs EntitySetName
The EntityName property defines the singular form used in metadata, while EntitySetName defines the plural form used in URLs. Both properties must be carefully coordinated.

### Backward Compatibility
Entity name changes break existing integrations. Plan naming carefully from initial implementation to avoid forced breaking changes in future versions.

### Regional Considerations
Entity names should use English terminology regardless of BC deployment region to maintain consistency across global implementations.

## URL Pattern Validation

### Discoverability Testing
Verify that entity URLs are discoverable through the standard OData metadata endpoints and follow expected RESTful resource patterns.

### Integration Impact Assessment
Consider how entity naming affects client code generation tools and SDK implementations that consume your APIs.

### Documentation Alignment
Ensure entity names align with business terminology used in API documentation and integration guides.

## Common Naming Pitfalls

### Inconsistent Pluralization
Mixing singular and plural forms across related entities creates confusion and breaks client expectations about resource collections.

### Technical vs Business Names
Using internal technical names instead of business-friendly terms reduces API adoption and increases integration complexity.

### Overly Generic Names
Names like "data" or "records" provide no semantic meaning and make APIs difficult to understand and maintain.

## Best Practices

### Domain-Driven Naming
Align entity names with business domain terminology to improve API usability and reduce the learning curve for integrators.

### Future-Proof Patterns
Choose names that can accommodate business evolution without requiring breaking changes to maintain API contract stability.

### Consistent Conventions
Establish and document naming conventions across all API entities within your solution to create predictable integration patterns.

### Validation Rules
Implement automated validation to ensure all API entity names follow established patterns and conventions before publication.