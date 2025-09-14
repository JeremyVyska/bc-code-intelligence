# Execution Validation - Quick Reference

## 📚 NEED MORE DETAIL?
For detailed validation commands and examples: `load_methodology execution-validation-full`

## 🎯 OBJECTIVE
Real-time fraud detection - verify claimed optimizations actually exist in code.

## ⚡ CORE VALIDATION (Runs every 2-3 files)
1. **File Content Check**: `grep -l "CalcSums" file1.al file2.al` - Must find claimed patterns
2. **File Modification**: `ls -la *.al | grep "$(date +%Y-%m-%d)"` - Files must show recent changes
3. **Pattern Quality**: `grep -A2 "CalcSums" file.al` - Check proper implementation
4. **Anti-Pattern Removal**: `grep -c "repeat" file.al` - Verify manual loops eliminated

## 🚨 FRAUD DETECTION (BLOCKING)
- **FALSE CLAIMS**: Optimization claimed but grep returns empty = STOP WORK
- **PHANTOM FILES**: Claims about unmodified files = STOP WORK  
- **INCORRECT PATTERNS**: Patterns exist but wrong implementation = FIX NOW
- **INCOMPLETE WORK**: Claims "complete" but evidence shows partial = STOP WORK

## 🔧 RESPONSE PROTOCOL
1. **STOP ALL WORK** until validation passes
2. **FORCE CORRECTION** of false claims  
3. **RE-VALIDATE** after correction
4. **DOCUMENT FRAUD** in optimization log

## ✅ SUCCESS CRITERIA
All claimed optimizations found in files + Recent modification timestamps + Correct pattern implementation + No false claims

## ➡️ RESULT
Pass = Continue work | Fail = BLOCK until corrected