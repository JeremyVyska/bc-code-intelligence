# Case Statement Multiple Conditions and Ranges - Code Samples

## Multiple Value Conditions

```al
codeunit 50102 "Case Multiple Conditions"
{
    // Example 1: Multiple discrete values for same action
    procedure GetBusinessHoursMessage(DayOfWeek: Integer; Hour: Integer): Text
    begin
        case DayOfWeek of
            1, 2, 3, 4, 5:  // Monday through Friday
                case Hour of
                    8, 9, 10, 11, 12, 13, 14, 15, 16, 17:  // 8 AM to 5 PM
                        exit('We are open - business hours');
                    18, 19:  // 6 PM to 7 PM
                        exit('Extended hours - limited service');
                    else
                        exit('We are closed - business day but outside hours');
                end;
            6:  // Saturday
                case Hour of
                    9, 10, 11, 12, 13:  // 9 AM to 1 PM
                        exit('Saturday hours - limited service');
                    else
                        exit('We are closed - Saturday outside hours');
                end;
            7:  // Sunday
                exit('We are closed - Sunday');
        end;
    end;

    // Example 2: Multiple country codes for regional processing
    procedure GetRegionalTaxRate(CountryCode: Code[10]): Decimal
    begin
        case CountryCode of
            'US', 'CA', 'MX':  // North America
                exit(CalculateNorthAmericaTax());
            'GB', 'FR', 'DE', 'IT', 'ES', 'NL', 'BE':  // Western Europe
                exit(CalculateEUTax());
            'JP', 'CN', 'KR', 'SG', 'AU':  // Asia Pacific
                exit(CalculateAsiaPacificTax());
            'BR', 'AR', 'CL', 'CO':  // South America
                exit(CalculateSouthAmericaTax());
            else
                exit(CalculateDefaultTax());
        end;
    end;

    // Example 3: Multiple product categories with shared logic
    procedure ApplyCategoryDiscount(ItemCategoryCode: Code[20]): Decimal
    var
        DiscountPercent: Decimal;
    begin
        case ItemCategoryCode of
            'ELECTRONICS', 'COMPUTERS', 'SOFTWARE':
                DiscountPercent := 5.0;  // Technology discount
            'CLOTHING', 'SHOES', 'ACCESSORIES':
                DiscountPercent := 15.0;  // Fashion discount
            'BOOKS', 'EDUCATION', 'TRAINING':
                DiscountPercent := 10.0;  // Educational discount
            'FOOD', 'BEVERAGE', 'ORGANIC':
                DiscountPercent := 8.0;  // Food & beverage discount
            else
                DiscountPercent := 0.0;  // No category discount
        end;
        exit(DiscountPercent);
    end;

    local procedure CalculateNorthAmericaTax(): Decimal
    begin
        exit(7.5);  // Example rate
    end;

    local procedure CalculateEUTax(): Decimal
    begin
        exit(20.0);  // Example VAT rate
    end;

    local procedure CalculateAsiaPacificTax(): Decimal
    begin
        exit(10.0);  // Example rate
    end;

    local procedure CalculateSouthAmericaTax(): Decimal
    begin
        exit(12.0);  // Example rate
    end;

    local procedure CalculateDefaultTax(): Decimal
    begin
        exit(5.0);  // Default rate
    end;
}
```

## Range-Based Conditions

```al
codeunit 50103 "Case Range Conditions"
{
    // Example 1: Quantity-based pricing tiers
    procedure CalculateVolumeDiscount(Quantity: Decimal): Decimal
    var
        DiscountPercent: Decimal;
    begin
        case Quantity of
            0:
                DiscountPercent := 0.0;  // No discount for zero quantity
            1..9:
                DiscountPercent := 0.0;  // No volume discount
            10..49:
                DiscountPercent := 5.0;  // Small volume discount
            50..99:
                DiscountPercent := 10.0; // Medium volume discount
            100..499:
                DiscountPercent := 15.0; // Large volume discount
            500..999:
                DiscountPercent := 20.0; // Enterprise discount
            else
                DiscountPercent := 25.0; // Maximum discount for 1000+
        end;
        exit(DiscountPercent);
    end;

    // Example 2: Amount-based shipping calculations
    procedure CalculateShippingCost(OrderAmount: Decimal; Weight: Decimal): Decimal
    var
        ShippingCost: Decimal;
    begin
        // First check order amount for free shipping
        case OrderAmount of
            100..999.99:
                ShippingCost := CalculateWeightBasedShipping(Weight) * 0.5; // 50% discount
            1000..9999.99:
                ShippingCost := 0; // Free shipping
            else
                ShippingCost := CalculateWeightBasedShipping(Weight); // Full rate
        end;
        
        exit(ShippingCost);
    end;

    // Example 3: Date-based seasonal pricing
    procedure GetSeasonalMultiplier(OrderDate: Date): Decimal
    var
        Month: Integer;
        Multiplier: Decimal;
    begin
        Month := Date2DMY(OrderDate, 2); // Extract month
        
        case Month of
            12, 1, 2:  // Winter months
                Multiplier := 1.1;  // 10% winter surcharge
            3, 4, 5:   // Spring months
                Multiplier := 1.0;  // Standard pricing
            6, 7, 8:   // Summer months
                Multiplier := 1.15; // 15% peak season surcharge
            9, 10, 11: // Fall months
                Multiplier := 0.95; // 5% off-season discount
        end;
        
        exit(Multiplier);
    end;

    // Example 4: Age-based service categories
    procedure DetermineServiceLevel(CustomerAge: Integer): Text
    begin
        case CustomerAge of
            0..17:
                exit('Youth Service - Parental approval required');
            18..24:
                exit('Student Service - Education discounts available');
            25..54:
                exit('Standard Service - Full access');
            55..64:
                exit('Pre-Senior Service - Early retirement benefits');
            65..150:
                exit('Senior Service - Senior citizen discounts');
            else
                exit('Invalid Age - Please verify information');
        end;
    end;

    // Example 5: Complex range conditions with business logic
    procedure AssignSalesTerritory(PostalCode: Code[10]): Code[20]
    var
        NumericCode: Integer;
    begin
        // Convert postal code to numeric for range evaluation
        if Evaluate(NumericCode, CopyStr(PostalCode, 1, 5)) then
            case NumericCode of
                00001..09999:
                    exit('NORTHEAST');
                10000..19999:
                    exit('MID_ATLANTIC');
                20000..29999:
                    exit('SOUTHEAST');
                30000..39999:
                    exit('SOUTH_CENTRAL');
                40000..49999:
                    exit('MIDWEST');
                50000..59999:
                    exit('MOUNTAIN_WEST');
                60000..79999:
                    exit('SOUTHWEST');
                80000..99999:
                    exit('WEST_COAST');
                else
                    exit('INTERNATIONAL');
            end
        else
            exit('UNKNOWN'); // Non-numeric postal codes
    end;

    local procedure CalculateWeightBasedShipping(Weight: Decimal): Decimal
    begin
        case Weight of
            0..1:
                exit(5.99);
            1.01..5:
                exit(9.99);
            5.01..10:
                exit(14.99);
            10.01..25:
                exit(19.99);
            else
                exit(29.99);
        end;
    end;
}
```

## Advanced Multiple Condition Patterns

```al
codeunit 50104 "Advanced Case Patterns"
{
    // Example 1: Combining multiple conditions with calculated values
    procedure DetermineCustomerRisk(CreditLimit: Decimal; DaysOverdue: Integer; PaymentHistory: Text): Text
    var
        RiskScore: Integer;
    begin
        // Calculate composite risk score
        RiskScore := 0;
        
        case CreditLimit of
            0..999:
                RiskScore += 3;     // High risk - low credit limit
            1000..4999:
                RiskScore += 2;     // Medium risk
            5000..19999:
                RiskScore += 1;     // Low risk
            else
                RiskScore += 0;     // Lowest risk - high credit limit
        end;
        
        case DaysOverdue of
            0:
                RiskScore += 0;     // No overdue amount
            1..30:
                RiskScore += 1;     // Slightly overdue
            31..60:
                RiskScore += 2;     // Moderately overdue
            61..90:
                RiskScore += 3;     // Seriously overdue
            else
                RiskScore += 4;     // Severely overdue
        end;
        
        case PaymentHistory of
            'EXCELLENT', 'GOOD':
                RiskScore -= 1;     // Reduce risk for good history
            'POOR', 'TERRIBLE':
                RiskScore += 2;     // Increase risk for poor history
        end;
        
        // Final risk assessment based on composite score
        case RiskScore of
            0..2:
                exit('LOW_RISK');
            3..4:
                exit('MEDIUM_RISK');
            5..7:
                exit('HIGH_RISK');
            else
                exit('EXTREME_RISK');
        end;
    end;

    // Example 2: Multiple condition matching with enum combinations
    procedure GetApprovalWorkflow(Amount: Decimal; Department: Option Finance,Sales,Operations,IT): Code[20]
    var
        AmountTier: Integer;
        WorkflowCode: Code[20];
    begin
        // Determine amount tier
        case Amount of
            0..999.99:
                AmountTier := 1;
            1000..4999.99:
                AmountTier := 2;
            5000..24999.99:
                AmountTier := 3;
            else
                AmountTier := 4;
        end;
        
        // Determine workflow based on department and amount tier
        case Department of
            Department::Finance:
                case AmountTier of
                    1: WorkflowCode := 'FIN_STANDARD';
                    2: WorkflowCode := 'FIN_MANAGER';
                    3: WorkflowCode := 'FIN_DIRECTOR';
                    4: WorkflowCode := 'FIN_CFO';
                end;
            Department::Sales:
                case AmountTier of
                    1, 2: WorkflowCode := 'SALES_STANDARD';
                    3: WorkflowCode := 'SALES_MANAGER';
                    4: WorkflowCode := 'SALES_VP';
                end;
            Department::Operations:
                case AmountTier of
                    1: WorkflowCode := 'OPS_AUTO';
                    2, 3: WorkflowCode := 'OPS_MANAGER';
                    4: WorkflowCode := 'OPS_DIRECTOR';
                end;
            Department::IT:
                case AmountTier of
                    1, 2: WorkflowCode := 'IT_STANDARD';
                    3, 4: WorkflowCode := 'IT_DIRECTOR';
                end;
        end;
        
        exit(WorkflowCode);
    end;

    // Example 3: Dynamic range calculations
    procedure CalculateDynamicPricing(BasePrice: Decimal; CustomerType: Code[10]; OrderDate: Date): Decimal
    var
        PriceMultiplier: Decimal;
        DaysFromToday: Integer;
        FinalPrice: Decimal;
    begin
        PriceMultiplier := 1.0; // Start with base multiplier
        DaysFromToday := OrderDate - Today;
        
        // Customer type adjustments
        case CustomerType of
            'GOLD', 'PLATINUM':
                PriceMultiplier *= 0.85; // 15% discount
            'SILVER', 'PREFERRED':
                PriceMultiplier *= 0.90; // 10% discount
            'BRONZE', 'STANDARD':
                PriceMultiplier *= 1.0;  // No discount
            'NEW', 'TRIAL':
                PriceMultiplier *= 1.05; // 5% markup
            else
                PriceMultiplier *= 1.10; // 10% markup for unknown types
        end;
        
        // Delivery date adjustments
        case DaysFromToday of
            -999..-1:    // Rush orders (past due)
                PriceMultiplier *= 1.25; // 25% rush charge
            0..7:        // Within a week
                PriceMultiplier *= 1.10; // 10% expedite charge
            8..30:       // Standard delivery
                PriceMultiplier *= 1.0;  // No adjustment
            31..90:      // Future delivery
                PriceMultiplier *= 0.95; // 5% discount for advance orders
            else         // Very far future
                PriceMultiplier *= 0.90; // 10% discount for long advance orders
        end;
        
        FinalPrice := BasePrice * PriceMultiplier;
        exit(FinalPrice);
    end;
}
```

Related atomic topic: Case Statement Multiple Conditions and Ranges
Knowledge reference: al-fundamentals-index.json#case-statements-advanced