# DeleteAll Method - Performance Analysis and Optimization

## Performance Comparison: DeleteAll vs Individual Deletes
```al
codeunit 50300 "Deletion Performance Analyzer"
{
    procedure CompareDeleteMethods(RecordCount: Integer)
    var
        TestData: Record "Performance Test Data";
        StartTime: DateTime;
        EndTime: DateTime;
        DeleteAllDuration: Duration;
        IndividualDeleteDuration: Duration;
    begin
        // Create test data
        CreateTestData(TestData, RecordCount);
        
        // Method 1: DeleteAll Performance Test
        StartTime := CurrentDateTime;
        TestData.Reset();
        TestData.SetRange("Test Batch", 1);
        TestData.DeleteAll(true);
        EndTime := CurrentDateTime;
        DeleteAllDuration := EndTime - StartTime;
        
        // Recreate test data for second test
        CreateTestData(TestData, RecordCount);
        
        // Method 2: Individual Delete Performance Test  
        StartTime := CurrentDateTime;
        TestData.Reset();
        TestData.SetRange("Test Batch", 1);
        if TestData.FindSet() then
            repeat
                TestData.Delete(true);  // Individual deletion with trigger
            until TestData.Next() = 0;
        EndTime := CurrentDateTime;
        IndividualDeleteDuration := EndTime - StartTime;
        
        // Report performance comparison
        Message('Performance Comparison for %1 records:\\' +
                'DeleteAll: %2 ms\\' +
                'Individual Deletes: %3 ms\\' +
                'Performance Gain: %4x faster',
                RecordCount,
                DeleteAllDuration,
                IndividualDeleteDuration,
                Round(IndividualDeleteDuration / DeleteAllDuration, 0.01));
    end;
}
```

## Performance Best Practices Summary

### 1. Batch Size Optimization
- **Small datasets (< 1,000 records)**: Single DeleteAll operation
- **Medium datasets (1,000-10,000)**: Single transaction acceptable  
- **Large datasets (> 10,000)**: Batch processing with commit points

### 2. Memory Management
- Monitor memory usage for large deletion operations
- Use smaller batch sizes in memory-constrained environments
- Force garbage collection between batches when needed

### 3. Transaction Strategy
- Keep transactions under 5,000 records for optimal performance
- Use commit points to prevent transaction log overflow
- Implement timeout handling for lock acquisition