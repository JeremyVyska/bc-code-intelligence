# Generic Method Patterns in AL - Code Examples

This sample demonstrates implementing generic method patterns for type-safe, reusable operations in AL.

## Basic Generic Interface

```al
// Generic interface for comparable operations
interface IComparable
{
    procedure CompareTo(Other: Interface IComparable): Integer;
}

// Generic interface for collection operations
interface IGenericCollection
{
    procedure Add(Item: Variant);
    procedure Remove(Item: Variant): Boolean;
    procedure Contains(Item: Variant): Boolean;
    procedure Count(): Integer;
    procedure Clear();
    procedure ToArray(): Array of [Variant];
}
```

## Generic List Implementation

```al
// Generic list implementation with type safety
codeunit 50800 "Generic List Manager"
{
    var
        Items: List of [Variant];
        ItemType: Text[50];

    procedure Initialize(ExpectedType: Text[50])
    begin
        Clear(Items);
        ItemType := ExpectedType;
    end;

    procedure Add(Item: Variant): Boolean
    begin
        if not ValidateItemType(Item) then
            exit(false);

        Items.Add(Item);
        exit(true);
    end;

    procedure AddRange(NewItems: List of [Variant]): Integer
    var
        Item: Variant;
        AddedCount: Integer;
    begin
        AddedCount := 0;
        foreach Item in NewItems do begin
            if Add(Item) then
                AddedCount += 1;
        end;

        exit(AddedCount);
    end;

    procedure Get(Index: Integer): Variant
    var
        EmptyVariant: Variant;
    begin
        if (Index < 1) or (Index > Items.Count) then
            exit(EmptyVariant);

        exit(Items.Get(Index));
    end;

    procedure Set(Index: Integer; Item: Variant): Boolean
    begin
        if not ValidateItemType(Item) then
            exit(false);

        if (Index < 1) or (Index > Items.Count) then
            exit(false);

        Items.Set(Index, Item);
        exit(true);
    end;

    procedure Remove(Item: Variant): Boolean
    var
        Index: Integer;
    begin
        Index := IndexOf(Item);
        if Index > 0 then begin
            Items.RemoveAt(Index);
            exit(true);
        end;

        exit(false);
    end;

    procedure RemoveAt(Index: Integer): Boolean
    begin
        if (Index < 1) or (Index > Items.Count) then
            exit(false);

        Items.RemoveAt(Index);
        exit(true);
    end;

    procedure IndexOf(Item: Variant): Integer
    var
        i: Integer;
        CurrentItem: Variant;
    begin
        for i := 1 to Items.Count do begin
            CurrentItem := Items.Get(i);
            if AreEqual(CurrentItem, Item) then
                exit(i);
        end;

        exit(0);
    end;

    procedure Contains(Item: Variant): Boolean
    begin
        exit(IndexOf(Item) > 0);
    end;

    procedure Count(): Integer
    begin
        exit(Items.Count);
    end;

    procedure Clear()
    begin
        Items.Clear();
    end;

    procedure ToArray(): List of [Variant]
    begin
        exit(Items);
    end;

    procedure Filter(FilterFunction: Text): List of [Variant]
    var
        FilteredItems: List of [Variant];
        Item: Variant;
    begin
        foreach Item in Items do begin
            if ApplyFilter(Item, FilterFunction) then
                FilteredItems.Add(Item);
        end;

        exit(FilteredItems);
    end;

    procedure Map(MapFunction: Text): List of [Variant]
    var
        MappedItems: List of [Variant];
        Item: Variant;
        MappedItem: Variant;
    begin
        foreach Item in Items do begin
            MappedItem := ApplyMap(Item, MapFunction);
            MappedItems.Add(MappedItem);
        end;

        exit(MappedItems);
    end;

    procedure Sort(): Boolean
    begin
        exit(SortItems(Items));
    end;

    procedure SortBy(CompareFunction: Text): Boolean
    begin
        exit(SortItemsBy(Items, CompareFunction));
    end;

    local procedure ValidateItemType(Item: Variant): Boolean
    var
        ActualType: Text[50];
    begin
        if ItemType = '' then
            exit(true); // No type constraint

        ActualType := GetVariantType(Item);
        exit(ActualType = ItemType);
    end;

    local procedure GetVariantType(Item: Variant): Text[50]
    begin
        case true of
            Item.IsText:
                exit('Text');
            Item.IsInteger:
                exit('Integer');
            Item.IsDecimal:
                exit('Decimal');
            Item.IsBoolean:
                exit('Boolean');
            Item.IsDate:
                exit('Date');
            Item.IsTime:
                exit('Time');
            Item.IsDateTime:
                exit('DateTime');
            Item.IsCode:
                exit('Code');
            else
                exit('Unknown');
        end;
    end;

    local procedure AreEqual(Item1: Variant; Item2: Variant): Boolean
    begin
        // Implement type-specific equality comparison
        if GetVariantType(Item1) <> GetVariantType(Item2) then
            exit(false);

        exit(Format(Item1) = Format(Item2));
    end;

    local procedure ApplyFilter(Item: Variant; FilterFunction: Text): Boolean
    begin
        // Simplified filter implementation
        case FilterFunction of
            'NOT_EMPTY':
                exit(Format(Item) <> '');
            'POSITIVE':
                begin
                    if Item.IsInteger then
                        exit(Item > 0);
                    if Item.IsDecimal then
                        exit(Item > 0);
                    exit(false);
                end;
            'NOT_ZERO':
                begin
                    if Item.IsInteger then
                        exit(Item <> 0);
                    if Item.IsDecimal then
                        exit(Item <> 0);
                    exit(true);
                end;
            else
                exit(true);
        end;
    end;

    local procedure ApplyMap(Item: Variant; MapFunction: Text): Variant
    var
        MappedValue: Variant;
        TextValue: Text;
        IntValue: Integer;
        DecValue: Decimal;
    begin
        case MapFunction of
            'TO_UPPER':
                begin
                    if Item.IsText or Item.IsCode then begin
                        TextValue := UpperCase(Format(Item));
                        MappedValue := TextValue;
                    end else
                        MappedValue := Item;
                end;
            'TO_LOWER':
                begin
                    if Item.IsText or Item.IsCode then begin
                        TextValue := LowerCase(Format(Item));
                        MappedValue := TextValue;
                    end else
                        MappedValue := Item;
                end;
            'DOUBLE':
                begin
                    if Item.IsInteger then begin
                        IntValue := Item * 2;
                        MappedValue := IntValue;
                    end else if Item.IsDecimal then begin
                        DecValue := Item * 2;
                        MappedValue := DecValue;
                    end else
                        MappedValue := Item;
                end;
            else
                MappedValue := Item;
        end;

        exit(MappedValue);
    end;

    local procedure SortItems(var ItemList: List of [Variant]): Boolean
    var
        i, j: Integer;
        temp: Variant;
        swapped: Boolean;
    begin
        try
            // Bubble sort implementation
            for i := 1 to ItemList.Count - 1 do begin
                swapped := false;
                for j := 1 to ItemList.Count - i do begin
                    if CompareVariants(ItemList.Get(j), ItemList.Get(j + 1)) > 0 then begin
                        temp := ItemList.Get(j);
                        ItemList.Set(j, ItemList.Get(j + 1));
                        ItemList.Set(j + 1, temp);
                        swapped := true;
                    end;
                end;
                if not swapped then
                    break;
            end;

            exit(true);

        except
            exit(false);
        end;
    end;

    local procedure SortItemsBy(var ItemList: List of [Variant]; CompareFunction: Text): Boolean
    begin
        // Custom sorting based on compare function
        exit(SortItems(ItemList)); // Simplified
    end;

    local procedure CompareVariants(Item1: Variant; Item2: Variant): Integer
    var
        Type1, Type2: Text[50];
        Text1, Text2: Text;
        Int1, Int2: Integer;
        Dec1, Dec2: Decimal;
        Date1, Date2: Date;
    begin
        Type1 := GetVariantType(Item1);
        Type2 := GetVariantType(Item2);

        if Type1 <> Type2 then begin
            Text1 := Format(Item1);
            Text2 := Format(Item2);
            case true of
                Text1 < Text2:
                    exit(-1);
                Text1 > Text2:
                    exit(1);
                else
                    exit(0);
            end;
        end;

        case Type1 of
            'Integer':
                begin
                    Int1 := Item1;
                    Int2 := Item2;
                    case true of
                        Int1 < Int2:
                            exit(-1);
                        Int1 > Int2:
                            exit(1);
                        else
                            exit(0);
                    end;
                end;
            'Decimal':
                begin
                    Dec1 := Item1;
                    Dec2 := Item2;
                    case true of
                        Dec1 < Dec2:
                            exit(-1);
                        Dec1 > Dec2:
                            exit(1);
                        else
                            exit(0);
                    end;
                end;
            'Date':
                begin
                    Date1 := Item1;
                    Date2 := Item2;
                    case true of
                        Date1 < Date2:
                            exit(-1);
                        Date1 > Date2:
                            exit(1);
                        else
                            exit(0);
                    end;
                end;
            else
                begin
                    Text1 := Format(Item1);
                    Text2 := Format(Item2);
                    case true of
                        Text1 < Text2:
                            exit(-1);
                        Text1 > Text2:
                            exit(1);
                        else
                            exit(0);
                    end;
                end;
        end;
    end;
}
```

## Generic Dictionary Implementation

```al
// Generic dictionary with type-safe key-value operations
codeunit 50801 "Generic Dictionary Manager"
{
    var
        Keys: List of [Variant];
        Values: List of [Variant];
        KeyType: Text[50];
        ValueType: Text[50];

    procedure Initialize(ExpectedKeyType: Text[50]; ExpectedValueType: Text[50])
    begin
        Clear(Keys);
        Clear(Values);
        KeyType := ExpectedKeyType;
        ValueType := ExpectedValueType;
    end;

    procedure Set(Key: Variant; Value: Variant): Boolean
    var
        ExistingIndex: Integer;
    begin
        if not ValidateKeyType(Key) or not ValidateValueType(Value) then
            exit(false);

        ExistingIndex := FindKeyIndex(Key);
        if ExistingIndex > 0 then begin
            Values.Set(ExistingIndex, Value);
        end else begin
            Keys.Add(Key);
            Values.Add(Value);
        end;

        exit(true);
    end;

    procedure Get(Key: Variant; var Value: Variant): Boolean
    var
        KeyIndex: Integer;
    begin
        KeyIndex := FindKeyIndex(Key);
        if KeyIndex > 0 then begin
            Value := Values.Get(KeyIndex);
            exit(true);
        end;

        Clear(Value);
        exit(false);
    end;

    procedure TryGet(Key: Variant): Variant
    var
        Value: Variant;
        EmptyVariant: Variant;
    begin
        if Get(Key, Value) then
            exit(Value)
        else
            exit(EmptyVariant);
    end;

    procedure ContainsKey(Key: Variant): Boolean
    begin
        exit(FindKeyIndex(Key) > 0);
    end;

    procedure ContainsValue(Value: Variant): Boolean
    var
        StoredValue: Variant;
        i: Integer;
    begin
        for i := 1 to Values.Count do begin
            StoredValue := Values.Get(i);
            if AreEqual(StoredValue, Value) then
                exit(true);
        end;

        exit(false);
    end;

    procedure Remove(Key: Variant): Boolean
    var
        KeyIndex: Integer;
    begin
        KeyIndex := FindKeyIndex(Key);
        if KeyIndex > 0 then begin
            Keys.RemoveAt(KeyIndex);
            Values.RemoveAt(KeyIndex);
            exit(true);
        end;

        exit(false);
    end;

    procedure Clear()
    begin
        Keys.Clear();
        Values.Clear();
    end;

    procedure Count(): Integer
    begin
        exit(Keys.Count);
    end;

    procedure GetKeys(): List of [Variant]
    begin
        exit(Keys);
    end;

    procedure GetValues(): List of [Variant]
    begin
        exit(Values);
    end;

    procedure GetKeyValuePairs(): List of [Dictionary of [Text, Variant]]
    var
        Pairs: List of [Dictionary of [Text, Variant]];
        Pair: Dictionary of [Text, Variant];
        i: Integer;
    begin
        for i := 1 to Keys.Count do begin
            Clear(Pair);
            Pair.Set('Key', Keys.Get(i));
            Pair.Set('Value', Values.Get(i));
            Pairs.Add(Pair);
        end;

        exit(Pairs);
    end;

    local procedure FindKeyIndex(Key: Variant): Integer
    var
        i: Integer;
        StoredKey: Variant;
    begin
        for i := 1 to Keys.Count do begin
            StoredKey := Keys.Get(i);
            if AreEqual(StoredKey, Key) then
                exit(i);
        end;

        exit(0);
    end;

    local procedure ValidateKeyType(Key: Variant): Boolean
    var
        ActualType: Text[50];
    begin
        if KeyType = '' then
            exit(true);

        ActualType := GetVariantType(Key);
        exit(ActualType = KeyType);
    end;

    local procedure ValidateValueType(Value: Variant): Boolean
    var
        ActualType: Text[50];
    begin
        if ValueType = '' then
            exit(true);

        ActualType := GetVariantType(Value);
        exit(ActualType = ValueType);
    end;

    local procedure GetVariantType(Item: Variant): Text[50]
    begin
        case true of
            Item.IsText:
                exit('Text');
            Item.IsInteger:
                exit('Integer');
            Item.IsDecimal:
                exit('Decimal');
            Item.IsBoolean:
                exit('Boolean');
            Item.IsDate:
                exit('Date');
            Item.IsTime:
                exit('Time');
            Item.IsDateTime:
                exit('DateTime');
            Item.IsCode:
                exit('Code');
            else
                exit('Unknown');
        end;
    end;

    local procedure AreEqual(Item1: Variant; Item2: Variant): Boolean
    begin
        if GetVariantType(Item1) <> GetVariantType(Item2) then
            exit(false);

        exit(Format(Item1) = Format(Item2));
    end;
}
```

## Generic Repository Pattern

```al
// Generic repository interface
interface IGenericRepository
{
    procedure Create(Entity: Dictionary of [Text, Variant]): Variant;
    procedure Read(Id: Variant): Dictionary of [Text, Variant];
    procedure Update(Id: Variant; Entity: Dictionary of [Text, Variant]): Boolean;
    procedure Delete(Id: Variant): Boolean;
    procedure FindAll(): List of [Dictionary of [Text, Variant]];
    procedure FindWhere(Criteria: Dictionary of [Text, Variant]): List of [Dictionary of [Text, Variant]];
}

// Generic repository implementation for customers
codeunit 50802 "Customer Repository" implements IGenericRepository
{
    procedure Create(Entity: Dictionary of [Text, Variant]): Variant
    var
        Customer: Record Customer;
        CustomerNo: Code[20];
        Name: Text[100];
        Address: Text[100];
    begin
        Customer.Init();
        Customer."No." := '';
        Customer.Insert(true);
        CustomerNo := Customer."No.";

        if Entity.Get('Name', Name) then
            Customer.Validate(Name, Name);

        if Entity.Get('Address', Address) then
            Customer.Validate(Address, Address);

        Customer.Modify(true);

        exit(CustomerNo);
    end;

    procedure Read(Id: Variant): Dictionary of [Text, Variant]
    var
        Customer: Record Customer;
        CustomerData: Dictionary of [Text, Variant];
        CustomerNo: Code[20];
    begin
        CustomerNo := Id;

        if Customer.Get(CustomerNo) then begin
            CustomerData.Set('CustomerNo', Customer."No.");
            CustomerData.Set('Name', Customer.Name);
            CustomerData.Set('Address', Customer.Address);
            CustomerData.Set('City', Customer.City);
            CustomerData.Set('PostCode', Customer."Post Code");
            CustomerData.Set('PhoneNo', Customer."Phone No.");
            CustomerData.Set('Email', Customer."E-Mail");
            CustomerData.Set('CreditLimit', Customer."Credit Limit (LCY)");
            CustomerData.Set('Blocked', Customer.Blocked);
        end;

        exit(CustomerData);
    end;

    procedure Update(Id: Variant; Entity: Dictionary of [Text, Variant]): Boolean
    var
        Customer: Record Customer;
        CustomerNo: Code[20];
        Name: Text[100];
        Address: Text[100];
        City: Text[30];
        PhoneNo: Text[30];
        Email: Text[80];
    begin
        CustomerNo := Id;

        if not Customer.Get(CustomerNo) then
            exit(false);

        try
            if Entity.Get('Name', Name) then
                Customer.Validate(Name, Name);

            if Entity.Get('Address', Address) then
                Customer.Validate(Address, Address);

            if Entity.Get('City', City) then
                Customer.Validate(City, City);

            if Entity.Get('PhoneNo', PhoneNo) then
                Customer.Validate("Phone No.", PhoneNo);

            if Entity.Get('Email', Email) then
                Customer.Validate("E-Mail", Email);

            Customer.Modify(true);
            exit(true);

        except
            exit(false);
        end;
    end;

    procedure Delete(Id: Variant): Boolean
    var
        Customer: Record Customer;
        CustomerNo: Code[20];
    begin
        CustomerNo := Id;

        if Customer.Get(CustomerNo) then begin
            Customer.Delete(true);
            exit(true);
        end;

        exit(false);
    end;

    procedure FindAll(): List of [Dictionary of [Text, Variant]]
    var
        Customer: Record Customer;
        CustomerList: List of [Dictionary of [Text, Variant]];
        CustomerData: Dictionary of [Text, Variant];
    begin
        if Customer.FindSet() then
            repeat
                Clear(CustomerData);
                CustomerData.Set('CustomerNo', Customer."No.");
                CustomerData.Set('Name', Customer.Name);
                CustomerData.Set('Address', Customer.Address);
                CustomerData.Set('City', Customer.City);
                CustomerList.Add(CustomerData);
            until Customer.Next() = 0;

        exit(CustomerList);
    end;

    procedure FindWhere(Criteria: Dictionary of [Text, Variant]): List of [Dictionary of [Text, Variant]]
    var
        Customer: Record Customer;
        CustomerList: List of [Dictionary of [Text, Variant]];
        CustomerData: Dictionary of [Text, Variant];
        CityFilter: Text[30];
        BlockedFilter: Boolean;
    begin
        if Criteria.Get('City', CityFilter) then
            Customer.SetRange(City, CityFilter);

        if Criteria.Get('Blocked', BlockedFilter) then
            Customer.SetRange(Blocked, BlockedFilter);

        if Customer.FindSet() then
            repeat
                Clear(CustomerData);
                CustomerData.Set('CustomerNo', Customer."No.");
                CustomerData.Set('Name', Customer.Name);
                CustomerData.Set('Address', Customer.Address);
                CustomerData.Set('City', Customer.City);
                CustomerData.Set('Blocked', Customer.Blocked);
                CustomerList.Add(CustomerData);
            until Customer.Next() = 0;

        exit(CustomerList);
    end;
}
```

## Usage Examples

```al
// Example of using generic patterns
codeunit 50803 "Generic Patterns Usage"
{
    procedure DemonstrateGenericList()
    var
        StringList: Codeunit "Generic List Manager";
        NumberList: Codeunit "Generic List Manager";
        Items: List of [Variant];
        FilteredItems: List of [Variant];
        Item: Variant;
        Count: Integer;
    begin
        // String list example
        StringList.Initialize('Text');
        StringList.Add('Apple');
        StringList.Add('Banana');
        StringList.Add('Cherry');

        Message('String list count: %1', StringList.Count());

        // Filter and map operations
        FilteredItems := StringList.Filter('NOT_EMPTY');
        FilteredItems := StringList.Map('TO_UPPER');

        foreach Item in FilteredItems do
            Message('Filtered item: %1', Item);

        // Number list example
        NumberList.Initialize('Integer');
        NumberList.Add(10);
        NumberList.Add(5);
        NumberList.Add(15);
        NumberList.Add(3);

        NumberList.Sort();
        Items := NumberList.ToArray();

        foreach Item in Items do
            Message('Sorted number: %1', Item);
    end;

    procedure DemonstrateGenericDictionary()
    var
        CustomerIndex: Codeunit "Generic Dictionary Manager";
        CustomerName: Variant;
        CustomerKeys: List of [Variant];
        Key: Variant;
    begin
        // Initialize dictionary with type constraints
        CustomerIndex.Initialize('Code', 'Text');

        // Add customer mappings
        CustomerIndex.Set('CUST001', 'Acme Corporation');
        CustomerIndex.Set('CUST002', 'Beta Industries');
        CustomerIndex.Set('CUST003', 'Gamma Solutions');

        // Retrieve values
        if CustomerIndex.Get('CUST001', CustomerName) then
            Message('Customer CUST001 is: %1', CustomerName);

        // Iterate through all keys
        CustomerKeys := CustomerIndex.GetKeys();
        foreach Key in CustomerKeys do begin
            CustomerName := CustomerIndex.TryGet(Key);
            Message('Customer %1: %2', Key, CustomerName);
        end;

        Message('Dictionary contains %1 customers', CustomerIndex.Count());
    end;

    procedure DemonstrateGenericRepository()
    var
        CustomerRepo: Codeunit "Customer Repository";
        CustomerData: Dictionary of [Text, Variant];
        AllCustomers: List of [Dictionary of [Text, Variant]];
        FilterCriteria: Dictionary of [Text, Variant];
        NewCustomerId: Variant;
        Customer: Dictionary of [Text, Variant];
    begin
        // Create new customer
        CustomerData.Set('Name', 'Generic Customer Ltd.');
        CustomerData.Set('Address', '123 Generic Street');
        CustomerData.Set('City', 'Generic City');

        NewCustomerId := CustomerRepo.Create(CustomerData);
        Message('Created customer with ID: %1', NewCustomerId);

        // Read customer back
        Customer := CustomerRepo.Read(NewCustomerId);
        Message('Retrieved customer: %1', Customer.Get('Name'));

        // Update customer
        Clear(CustomerData);
        CustomerData.Set('Address', '456 Updated Avenue');
        CustomerData.Set('PhoneNo', '555-0123');

        if CustomerRepo.Update(NewCustomerId, CustomerData) then
            Message('Customer updated successfully');

        // Find customers by criteria
        FilterCriteria.Set('City', 'Generic City');
        AllCustomers := CustomerRepo.FindWhere(FilterCriteria);
        Message('Found %1 customers in Generic City', AllCustomers.Count);
    end;
}
```

This comprehensive implementation demonstrates generic programming patterns in AL, providing type-safe, reusable operations for collections, dictionaries, and repositories while maintaining AL's strong typing benefits.