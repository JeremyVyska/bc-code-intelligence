# Code Analysis Phase - Systematic Discovery Methodology

## Overview
**Objective**: Comprehensive code analysis with systematic discovery to ensure complete coverage of all business modules and prevent missed optimization opportunities.

**Key Principle**: Different developers and AI agents often miss critical areas due to unsystematic exploration. This methodology ensures 100% coverage through structured analysis.

## 🔥 COMPACTION-RESISTANT CORE PROCESS (ALWAYS REMEMBER)
**For EVERY procedure in EVERY module:**
1. **Query ALL patterns**: `find_bc_topics` domain="all" → get complete pattern inventory
2. **Check EVERY pattern**: For each pattern found, run `analyze_code_patterns` on current procedure  
3. **Document coverage**: Record WHICH patterns were checked (proves completeness)
4. **Validate before next**: Confirm you checked ALL patterns before moving to next procedure

**This 4-step loop is the core systematic process that must survive context compression.**

---

## Phase Execution Process

### Step 1: Workspace Assessment & Inventory Management
**CRITICAL**: Establish systematic coverage to prevent missing critical files in large codebases.

#### 🔍 Phase 1A: Workspace Assessment
- [ ] **Complete File Inventory**
  - [ ] Scan entire project structure for all .al files
  - [ ] Count total files and categorize by type (codeunits, tables, pages, reports)
  - [ ] Assess codebase complexity:
    - Small: <50 files (standard analysis)
    - Medium: 50-200 files (focused analysis recommended)
    - Large: 200+ files (user scoping required)
    - Enterprise: 500+ files (significant time and token investment)

#### 🚨 Phase 1B: User Feedback Gate (Large Codebases)
**IF codebase contains 200+ files:**
- [ ] **Present scope options to user:**
  - A) Comprehensive analysis (all files - significant time/token cost)  
  - B) Module-focused analysis (specify business domains)
  - C) Performance-critical files only (reports, analytics, data processing)
  - D) Custom file selection (user-specified scope)
- [ ] **Wait for user scope selection before proceeding**
- [ ] **Document scope decision** in analysis log

#### 📋 Phase 1C: Coverage Checklist Creation  
Based on scope selection:
- [ ] **File Inventory Checklist**: Create comprehensive list of files to analyze
- [ ] **Pattern Detection Checklist**: List all optimization patterns to check per file
- [ ] **Expected Coverage Documentation**: "Analyzing [X] files across [Y] modules for [Z] patterns"
- [ ] **Progress Tracking**: Update checklists as analysis proceeds

#### 🔒 MANDATORY ENFORCEMENT GATES
**CRITICAL**: These gates CANNOT be bypassed - failing any gate BLOCKS progression to next phase.

- [ ] **Gate 1: Complete File Inventory**
  - [ ] Generate actual file list using `find` or `glob` commands (if within a Workspace, use the workspace folders. Otherwise, use the root folder.)
  - [ ] Cross-reference with planned analysis scope
  - [ ] **BLOCK ANALYSIS**: Cannot proceed until file inventory is verified complete
  
- [ ] **Gate 2: Systematic Coverage Validation**
  - [ ] Document EVERY file that will be analyzed
  - [ ] Verify EVERY pattern will be checked per file
  - [ ] **FAILURE CONDITION**: Missing ANY files from scope = analysis restart required

- [ ] **Gate 3: File-by-File Progress Tracking**  
  - [ ] Create explicit checklist: `[ ] filename.al - ANALYZED`
  - [ ] Mark each file as complete ONLY after analysis documented
  - [ ] **BLOCKING CONDITION**: Cannot claim module complete until ALL files marked analyzed
  - [ ] **EXAMPLE**: Analytics Module (4/4): ✅ T4_File1.al ✅ T4_File2.al ✅ T4_File3.al ✅ T4_File4.al

#### 🔍 Phase 1D: Business Domain Discovery
- [ ] **Scan Project Structure** (within defined scope)
  - [ ] Review all codeunit files in analysis scope
  - [ ] Identify distinct business domains by naming patterns
  - [ ] **CRITICAL**: Examine all table objects (.al files) to understand data model
  - [ ] **Include related objects**: If codeunits reference tables, analyze those table definitions too
  - [ ] **Cross-reference shared infrastructure**: Review any shared/common objects across project tiers
  
- [ ] **Categorize Business Functions**
  - [ ] Financial Operations (payments, invoicing, accounting)
  - [ ] Data Processing (calculations, aggregations, transformations)  
  - [ ] Reporting & Analytics (reports, dashboards, summaries)
  - [ ] Core Business Logic (workflows, validations, business rules)
  - [ ] Integration & API (external systems, data exchange)
  - [ ] User Interface (pages, actions, user interactions)
  - [ ] Background Processing (jobs, scheduled tasks, batch operations)
  - [ ] Data Maintenance (cleanup, archival, migration)
  - [ ] Security & Permissions (access control, audit trails)

- [ ] **Cross-Reference Analysis**
  - [ ] Map codeunits to business functions they serve
  - [ ] Identify shared utilities and common services
  - [ ] Document interdependencies between modules

### Step 2: Complete Procedure-Level Analysis
**CRITICAL**: For each discovered business module, analyze EVERY procedure systematically:

#### 🔍 Systematic Procedure Discovery Process
- [ ] **Enumerate All Procedures**
  - List EVERY procedure in each codeunit
  - Do not skip any procedures - even small ones may have optimization opportunities
  - Create complete procedure inventory before analysis begins

- [ ] **Individual Procedure Analysis** (COMPACTION-RESISTANT PROCESS)
  - **Step A**: For EACH procedure, run `find_bc_topics` with domain="all" to discover complete pattern inventory available
  - **Step B**: For EACH pattern from Step A, run `analyze_code_patterns` to check current procedure against that pattern
  - **Step C**: Document WHICH patterns were checked (not just what was found) - this proves systematic coverage
  - **Step D**: Before moving to next procedure, validate you checked ALL patterns from Step A inventory
  - **Step E**: Cross-check completeness: Ensure no procedures were skipped in any module
  - **Step F**: If patterns checked require checking related objects (tables, other codeunits), do so and document findings

#### 🚨 Critical Performance Anti-Patterns to Find (Per Procedure)
- [ ] **Manual Loop Processing**
  - Look for `repeat...until` loops processing records
  - Identify manual summation or aggregation operations
  - Find sequential record processing that could be optimized

- [ ] **Inefficient Database Access**
  - Multiple database calls in loops
  - Missing field loading optimizations
  - Inefficient filtering or sorting operations

- [ ] **Nested Loop Anti-Patterns**
  - **CRITICAL**: Look for procedures with nested record processing
  - Identify outer loop → inner loop patterns that create N+1 query problems
  - Find procedures that process parent records, then child records for each parent

- [ ] **Complete Pattern Library Coverage** (COMPACTION-RESISTANT)
  - **Pattern Discovery Query**: Use `find_bc_topics` with broad search terms to enumerate ALL available optimization patterns
  - **Systematic Cross-Reference**: For each procedure, check it against EVERY pattern in the discovered inventory  
  - **Coverage Documentation**: Record which pattern categories were checked for each procedure (proves no patterns were missed)
  - **Validation Loop**: Before analysis completion, verify every procedure was checked against every available pattern

### Step 3: Prioritization Framework
Rank discovered optimization opportunities:

#### 🎯 Priority Classification
- [ ] **🔴 Critical (Red)**: Performance killers affecting core operations
- [ ] **🟡 High (Yellow)**: Significant improvements with moderate effort
- [ ] **🟢 Medium (Green)**: Good improvements requiring analysis
- [ ] **⚪ Low (White)**: Minor optimizations or already optimized

#### 📊 Impact Assessment
For each optimization opportunity:
- [ ] Estimate performance impact (execution time reduction)
- [ ] Assess implementation complexity
- [ ] Consider business criticality of the affected process
- [ ] Evaluate risk level of proposed changes

### Step 4: Documentation Requirements (COMPACTION-RESISTANT VALIDATION)
- [ ] **Module Coverage Report**: List all business modules discovered
- [ ] **Complete Procedure Inventory**: Document EVERY procedure analyzed in EVERY module
- [ ] **Pattern Coverage Matrix**: Document WHICH pattern categories were checked for EACH procedure (proves systematic coverage)
- [ ] **Table Structure Analysis**: Document key table objects, SIFT keys, and FlowFields found
- [ ] **Procedure-Level Pattern Analysis**: Document anti-patterns found in EACH procedure
- [ ] **Coverage Validation Evidence**: Provide proof that no procedures were missed AND no patterns were missed
- [ ] **Cross-Infrastructure Analysis**: Note shared tables/infrastructure that could benefit multiple modules
- [ ] **Optimization Roadmap**: Prioritized list of improvement opportunities from ALL procedures
- [ ] **Knowledge Gaps**: Areas requiring additional platform expertise

---

## Success Criteria
✅ **Complete Coverage**: All business modules in codebase identified and analyzed
✅ **Procedure-Level Completeness**: EVERY procedure in EVERY module has been individually analyzed
✅ **Pattern Detection**: Performance anti-patterns documented with specific examples from each procedure
✅ **No Gaps**: Verification that no procedures were skipped during analysis
✅ **Prioritization**: Clear ranking of optimization opportunities by impact/effort
✅ **Actionable Output**: Concrete next steps for performance optimization phase

## Quality Validation (COMPACTION-RESISTANT CHECKS)
- **Module Thoroughness**: Can you explain the business purpose of every codeunit?
- **Procedure Completeness**: Have you analyzed every single procedure in every module?
- **Pattern Library Coverage**: Can you list WHICH pattern categories you checked for each procedure?
- **Systematic Validation**: Did you run `find_bc_topics` to discover ALL available patterns before analysis?
- **Coverage Evidence**: Can you prove no procedures were missed AND no patterns were missed?
- **Pattern Verification**: Have you found actual optimization opportunities, not just cosmetic issues?
- **Impact Assessment**: Are your priority rankings based on measurable performance impact?

---

## Next Phase
Upon completion, proceed to **Performance Optimization Phase** with prioritized list of improvements.