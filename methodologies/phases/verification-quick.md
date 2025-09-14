# Verification Phase - Quick Reference

## 📚 NEED MORE DETAIL?
For detailed validation procedures and examples: `load_methodology verification-full`

## 🎯 OBJECTIVE
Final coverage audit to catch missed files and validate optimization completeness.

## ⚡ CORE VERIFICATION

### 🚨 CONTEXT-RESISTANT FILE REDISCOVERY (MANDATORY FIRST STEP)
**CRITICAL**: Never trust chat memory or existing logs - always rediscover independently
```bash
# 1. Rediscover actual file scope in workspace
find . -name "*.al" | wc -l
find . -name "*.al" > actual-files.tmp

# 2. Count files claimed in optimization log
grep -c "\.al" OptimizationLog.md

# 3. BLOCKING CONDITION: Mismatched counts = INCOMPLETE WORK
```

### Standard Verification Steps
1. **File Coverage Audit**: Compare rediscovered files vs optimization log claims
2. **Phantom File Detection**: Check for claimed work on non-existent files  
3. **Missing File Detection**: Find actual files missing from optimization log
4. **Gap Resolution**: Address ALL discovered coverage gaps
5. **Accuracy Validation**: Cross-reference claims with actual file contents

## 🚨 COVERAGE VALIDATION
- **File Inventory vs Reality**: `find . -name "*.al"` vs analysis file list
- **Modification Evidence**: All claimed files must show recent timestamps
- **Pattern Verification**: `grep -l "CalcSums" *.al` must match optimization claims
- **Anti-Pattern Check**: Claimed eliminated patterns must be gone

## 🔧 GAP RESOLUTION PROTOCOL
1. **Identify gaps** in coverage (missed files, phantom claims)
2. **BLOCK completion** until all gaps resolved
3. **Return to analysis/performance** for missed files
4. **Re-validate** after gap resolution
5. **Document accuracy** in final report

## ✅ SUCCESS CRITERIA  
Zero phantom files + Zero missing files + 100% claimed optimizations verified + All gaps resolved + Complete accuracy validation

## 🏁 COMPLETION
Only mark complete when ALL verification checks pass - no exceptions