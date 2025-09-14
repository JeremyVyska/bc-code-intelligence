---
title: "FieldError Default Message Logic"
domain: "error-handling"
difficulty: "intermediate"
bc_versions: "14+"
tags: ["fielderror", "default-messages", "field-state", "error-text"]
samples: "samples/fielderror-default-messages.md"
---

# FieldError Default Message Logic

## Overview

When FieldError is called without the MsgText parameter, Business Central automatically generates contextual error messages based on field state and metadata. This default message logic provides consistent, informative error reporting with minimal code.

**Key Advantage**: Automatic message generation reduces code maintenance while providing rich contextual information.

## Default Message Construction

### Message Components
Business Central constructs default error messages using five key components:
1. **Field Caption**: From table field definition
2. **Current Value**: Field content (if not empty)
3. **Standard Text**: Context-appropriate default message
4. **Record Context**: Primary key fields for record identification
5. **Table Name**: Source table identification

### Field State Detection

#### Empty Field Messages
For empty fields, BC generates "must have a value" or "must be filled in" messages depending on field type. Code and Text fields use "must have a value" while Date fields use "must be filled in".

#### Non-Empty Field Messages
For fields containing data, BC generates "is not valid" messages that include the current field value, helping users identify exactly what data caused the validation failure.

#### Record Context Inclusion
All FieldError messages automatically include primary key field values to provide complete record identification, enabling users to locate the specific record causing the error.

## Default Message Patterns

### Standard Empty Field Pattern
Structure follows: "[Field Caption] must have a value in [Table Name] [Record Key]"
- **Use Case**: Required field validation
- **Common Scenario**: Setup tables, document headers, required configuration

### Standard Value Conflict Pattern
Structure follows: "[Field Caption] [Current Value] already exists in [Table Name] [Record Key]"
- **Use Case**: Uniqueness violations, duplicate detection
- **Common Scenario**: Primary key conflicts, unique index violations

### Context-Specific Patterns
Business Central applies different default messages based on:
- Field data type and properties
- Table relationships and constraints
- Current field state and validation context

## Strategic Decision Matrix

### Use Default Messages When
- **Standard validation scenarios**: Common empty/duplicate field errors
- **Consistent user experience desired**: Uniform error message formatting
- **Minimal maintenance required**: Automatic message updates with metadata changes
- **Field caption changes**: Messages automatically reflect caption updates

### Use Custom Messages When
- **Business rule violations**: Complex validation logic requiring explanation
- **Specific guidance needed**: Error messages should guide user resolution
- **Context-sensitive information**: Additional fields or calculations referenced
- **User experience optimization**: Tailored messages for specific user workflows

## Implementation Examples

### Leveraging Default Messages
Setup validation scenarios benefit from default messaging where standard "must have a value" text provides adequate user guidance without additional context.

### Mixed Default and Custom Messages
Combining default empty field detection with custom business logic provides optimal balance between consistency and specificity in error reporting.

### Record State-Aware Validation
Different validation approaches based on record state allow for appropriate use of default messages for standard scenarios while applying custom messages for complex business rules.

## Best Practices

### Default Message Optimization
- **Field captions**: Ensure field captions are user-friendly and descriptive
- **Table design**: Structure primary keys to provide meaningful record context
- **Consistent patterns**: Use default messages for standard validation scenarios
- **Message testing**: Verify default messages provide adequate user guidance

### Custom vs Default Decision Process
1. **Evaluate standard patterns**: Does default message adequately describe the problem?
2. **Consider user guidance**: Do users need additional context or resolution steps?
3. **Assess business rules**: Does the error involve complex business logic?
4. **Plan for maintenance**: Will custom messages require ongoing updates?

### Performance Considerations
- **Default message efficiency**: BC generates default messages efficiently
- **Custom message overhead**: Complex custom messages may impact performance
- **Field access patterns**: Default messages access field metadata, not additional data
- **Localization support**: Default messages automatically support multiple languages

## Message Quality Assessment

### Effective Default Messages
- **Clear field identification**: Field caption clearly identifies the problematic field
- **Sufficient context**: Record key provides adequate record identification
- **Consistent formatting**: Uniform message structure across application
- **Automatic updates**: Messages reflect metadata changes without code updates

### When Defaults Fall Short
- **Insufficient business context**: Generic messages don't explain business rules
- **Poor field captions**: Technical field names don't guide users effectively
- **Complex relationships**: Multi-field or cross-table validation scenarios
- **Specific user workflows**: Messages don't align with user task patterns

## Localization and Maintenance

### Automatic Benefits
- **Language support**: Default messages automatically support BC language packs
- **Metadata synchronization**: Messages update when field captions change
- **Consistent terminology**: Standard message patterns across entire application
- **Reduced translation overhead**: No custom message translation required

### Maintenance Advantages
- **Code simplification**: Fewer message strings to maintain
- **Automatic updates**: Metadata changes propagate to error messages
- **Quality consistency**: Standard message quality across all validation scenarios
- **Development efficiency**: Faster implementation of standard validation patterns

## Advanced Default Message Behavior

### Field Type Variations
Different field types generate contextually appropriate default messages. Boolean fields show "Yes/No" states, DateTime fields include full timestamp information, and GUID fields display the complete identifier.

### Multi-Language Considerations
Default messages automatically adapt to user language settings without requiring additional translation effort, making them ideal for international deployments.

### Record Key Complexity
Tables with complex primary keys automatically include all key fields in error context, providing comprehensive record identification regardless of table structure complexity.

## Implementation Guidelines

Default message logic works most effectively when field captions accurately represent user-facing terminology and primary key structures provide meaningful record identification. Strategic use of default messages reduces maintenance overhead while maintaining user experience quality.

For detailed implementation examples and working code samples demonstrating default message behavior across various field types and scenarios, see the companion AL code file.

