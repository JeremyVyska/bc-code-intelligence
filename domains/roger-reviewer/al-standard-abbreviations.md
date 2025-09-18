---
title: "AL Standard Abbreviations"
domain: "roger-reviewer"
difficulty: "beginner"
bc_versions: "14+"
tags: ["abbreviations", "naming", "standards", "reference"]
type: "improvement-pattern"
category: "code-quality"
pattern_type: "good"
severity: "high"
impact_level: "high"
improvement_suggestion: "Use only Microsoft-approved abbreviations to maintain consistency with Business Central platform"
implementation_steps: ["verify-platform-abbreviations", "apply-standard-patterns", "validate-consistency"]
validation_criteria: ["platform-compliant-abbreviations", "consistent-usage"]
technical_areas: ["al-code"]
---

# AL Standard Abbreviations

## Overview

Business Central uses standardized abbreviations throughout the platform for consistency and space efficiency. These abbreviations are deeply embedded in field names, variable naming, and API structures across all BC modules.

**Key Principle**: Use only established BC abbreviations to maintain platform consistency and avoid confusion with standard BC objects and APIs.

## Strategic Framework

### Core Business Abbreviations
- **No.** → Number (CustomerNo, ItemNo, DocumentNo)
- **Qty** → Quantity (BaseQty, OutstandingQty, ReservedQty)
- **Amt** → Amount (SalesAmt, CostAmt, DiscountAmt)
- **Desc** → Description (ItemDesc, CustomerDesc)
- **LCY** → Local Currency (AmountLCY, UnitCostLCY)
- **FCY** → Foreign Currency (AmountFCY, UnitPriceFCY)

### Financial and Accounting
- **G/L** → General Ledger (GLEntry, GLAccount)
- **VAT** → Value Added Tax (VATEntry, VATAmount)
- **A/R** → Accounts Receivable (ARAmount)
- **A/P** → Accounts Payable (APAmount)
- **WIP** → Work in Progress (WIPEntry)
- **COGS** → Cost of Goods Sold (COGSAmount)

### Operations and Inventory
- **UOM** → Unit of Measure (BaseUOM, SalesUOM)
- **BOM** → Bill of Materials (BOMComponent)
- **SKU** → Stock Keeping Unit (SKUCode)
- **Whse** → Warehouse (WhseEntry, WhseShipment)
- **Invt** → Inventory (InvtPostingGroup)
- **Prod** → Production (ProdOrder, ProdBOMLine)

## Architecture Focus

### Platform Integration
BC abbreviations ensure seamless integration with standard platform objects, APIs, and external systems expecting specific field naming patterns.

### Database Considerations
Abbreviations help manage field name length limits while maintaining readability. Critical for table and field names that interface with external systems.

### API Consistency
Standard abbreviations ensure API field names match platform conventions, maintaining compatibility with Microsoft and third-party integrations.

### Localization Support
Platform abbreviations support localization frameworks where translations may require different text lengths while maintaining field name consistency.

## Best Practices

### Approved Usage Only
- **Platform Standards**: Use only abbreviations established in BC base application
- **Consistency Check**: Verify abbreviations match existing BC field names
- **Avoid Custom**: Never create new abbreviations without platform precedent
- **Documentation**: When platform lacks precedent, document rationale and team approval

### Context-Appropriate Application
- **Field Names**: Essential for table field naming within character limits
- **Variable Names**: Use when matching related field names or platform objects
- **Procedure Names**: Generally avoid abbreviations in procedure names
- **Comments**: Spell out full terms in documentation and comments

### Validation Requirements
- **Platform Verification**: Cross-check against BC object definitions
- **API Compatibility**: Ensure abbreviations don't conflict with standard APIs
- **Extension Guidelines**: Follow AppSource and per-tenant extension naming requirements
- **Team Standards**: Maintain approved abbreviation list for project-specific terms

## Anti-Patterns

### Avoid These Approaches
- **Custom Creations**: Never invent abbreviations not used in BC platform
- **Inconsistent Forms**: Don't mix "Descr" and "Desc" for same concept
- **Ambiguous Terms**: Avoid abbreviations with multiple possible meanings
- **Platform Conflicts**: Don't use abbreviations that conflict with BC standards

Invalid abbreviations cause integration failures and platform inconsistencies, potentially leading to AppSource validation failures.

*Related patterns: al-variable-naming-conventions.md, al-field-naming.md*