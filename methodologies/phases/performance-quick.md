# Performance Phase - Quick Reference

## 📚 NEED MORE DETAIL?
For implementation examples and detailed guidance: `load_methodology performance-full`

## 🎯 OBJECTIVE
Apply systematic optimizations with real-time validation to prevent execution fraud.

## ⚡ CORE PROCESS
1. **Priority Order**: Database-native (CalcSums) → Platform-optimized (SetLoadFields) → Application-level (FlowFields)
2. **Infrastructure Check**: Update tables/keys to support optimizations (SIFT keys for CalcSums)
3. **Apply Optimizations**: Focus on high-impact patterns from analysis phase
4. **Execution Validation**: Run `load_methodology execution-validation` every 2-3 files

## 🚨 EXECUTION CHECKPOINTS (MANDATORY)
- **After 2-3 files**: `load_methodology execution-validation`
- **Before module claims**: BLOCK until validation passes
- **Before continuing**: Validate all claimed work exists
- **After optimization claims**: Verify CalcSums/SetLoadFields actually implemented

## 🔍 OPTIMIZATION HIERARCHY
**Tier 1**: `MyRecord.CalcSums(Amount)` - Database aggregation  
**Tier 2**: `MyRecord.SetLoadFields()` - Field optimization  
**Tier 3**: `MyRecord.CalcFields()` - FlowField calculations  

## 🚨 BLOCKING CONDITIONS
- **Optimization without infrastructure** = INVALID
- **Claims without file modifications** = FRAUD  
- **Incomplete coverage** of analysis scope = INCOMPLETE

## ➡️ NEXT PHASE
Execution validation → Verification phase for final coverage audit