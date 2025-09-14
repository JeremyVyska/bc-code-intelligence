# Analysis Phase - Quick Reference

## 📚 NEED MORE DETAIL?
For implementation examples and detailed guidance: `load_methodology analysis-full`

## 🎯 OBJECTIVE
Systematic pattern discovery across entire codebase with complete coverage validation.

## ⚡ CORE PROCESS
1. **Workspace Assessment**: Count files, estimate scope, get user approval for large codebases (100+ files)
2. **File Inventory**: Create complete checklist of all files to analyze
3. **Pattern Discovery**: For each procedure: `find_bc_topics domain="all"` → `analyze_code_patterns` 
4. **Coverage Tracking**: Mark each file `[ ] filename.al - ANALYZED` only after documentation complete
5. **Dependency Discovery**: After any changes, ask "Does this depend on other files?" - address immediately (Step F)

## 🚨 INVENTORY PRESERVATION RULE - CRITICAL
**NEVER OVERWRITE FILE INVENTORY SECTIONS** - Only update checkbox status:
- ❌ **FORBIDDEN**: Rewriting/removing file inventory lists
- ❌ **FORBIDDEN**: "Cleaning up" or reorganizing inventory sections  
- ❌ **FORBIDDEN**: Removing "pending" files from lists
- ✅ **ALLOWED**: Change `[ ]` to `[x]` for completed files only
- ✅ **ALLOWED**: Add new analysis content BELOW inventory sections

## 🚨 BLOCKING CONDITIONS
- **Cannot skip any files** in scope
- **Cannot claim module complete** until ALL files marked analyzed  
- **Must document which patterns checked** (not just what found)
- **Must validate completeness** before proceeding to performance phase

## 🔍 SUCCESS CRITERIA
✅ Complete file inventory created and validated  
✅ Every file has documented analysis with pattern coverage  
✅ All anti-patterns identified with specific locations  
✅ Related objects examined and documented  
✅ Zero files skipped or missed in analysis scope  

## 📋 KEY OUTPUTS
- File-by-file analysis documentation
- Anti-pattern inventory with locations
- Optimization opportunity prioritization
- Infrastructure gaps identified (missing SIFT keys, etc.)

## ➡️ NEXT PHASE
Performance optimization with execution validation checkpoints