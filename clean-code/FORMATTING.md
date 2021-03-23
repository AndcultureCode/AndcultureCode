# Clean Code: Formatting

[Table of Contents](../CLEAN-CODE.md)

## Vertical Formatting

### General

One way to think of our code is like reading a newspaper story: as we move downward, the amount of detail within our code should increase.

While there is no maximum file line length, if a file reaches 500 lines we should consider refactoring it into multiple files.
We want to keep our files small so that they are easier to understand.

Use blank lines to create space between thoughts, and group related code.
Unnecessary comments, such as those on well-named variables, creates space that prevents grouping.

### Ordering

While files should be kept small, we realize that due to varying factors files
extend well beyond 100 lines. Due to this reality, our team leverages regions to organize our
different members.

Please refer to the following documentation for regions by language:

-   [CSharp](https://github.com/AndcultureCode/AndcultureCode/blob/main/CODE-CONVENTIONS-CSHARP.md)
-   [TypeScript](https://github.com/AndcultureCode/AndcultureCode/blob/main/CODE-CONVENTIONS-TYPESCRIPT.md)

#### Variable / Property Regions

Regions containing properties should maintain alphabetical order.

```CSharp
#region Properties

public string Description { get; set; }
public bool IsActive { get; set; }
public string Name { get; set; }

#endregion Properties
```

#### Function / Method Regions

Regions containing functions should order members downward by call order. In other words, a
function that is called should be below a function that does the calling.

```CSharp
#region Public Methods

public void Import(long importId)
{
    var files = FindAll(importId);
    ImportFiles(files);
}

public IEnumerable<File> FindAll(long importId)
    => _fileReadConductor.FindAll(e => e.ImportId == importId).ToList();

public void ImportFiles(IEnumerable<File> files) => files.ForEach(ImportFile);

public void ImportFile(File file)
{
    // import logic
}

#endregion Public Methods
```

## Horizontal Formatting

### General

Strive to maintain a 100 character width maximum per line (shorter is typically better). Ideally,
this would be enforced by an auto-formatter or linting tools, but lines that are hard to read due to
length can also be called out in code reviews.

### Alignment

Horizontal alignment is nice to look at, but it can take away from the true intent of your code.

```CSharp
 var pasteValidationResult = _pasteValidateConductor.Validate(contentAreaVersionComponentId:       id,
                                                              courseId:                            dto.CourseId,
                                                              sourceContentAreaVersionComponentId: dto.SourceId);
```

If a list of declarations is more easily read when aligned horizontally, the chances are you should consider breaking up the list into another class, etc.

We highly encourage you and your team to consider the use of automated formatting tools to standardize and simplify your formatting practice. The below example utilizes the default Microsoft formatter for C#.

```CSharp
var pasteValidationResult = _pasteValidateConductor.Validate(
    contentAreaVersionComponentId: id,
    courseId: dto.CourseId,
    sourceContentAreaVersionComponentId: dto.SourceId
);
```

### Indentation

When encountering single lines of execution within a block of code, you should use curly braces.  Curly braces provide a visual aid in reading scoped code execution.  They also aid in preventing unexpected errors.

Consider the following code example which we consider a bad coding practice:
```CSharp
if (radius > upperLimit)
    return true;
else if (radius < lowerLimit)
    return true;
else
    return false;
```

Left to its current logical code execution paths, the above should be re-written like this:
```CSharp
if (radius > upperLimit)
{
    return true;
}
else if (radius < lowerLimit)
{
    return true;
}
else
{
    return false;
}
```

While this refactored code is easier to read, the inclusion of the `else if` and `else` further muddy its implementation and its desireability.  This logic could be inversed making for a cleaner, shorter set of code that accomplishes the same thing.  Check it out:
```CSharp
if (radius >= lowerLimit && radius <= upperLimit)
{
    return false;
}
return true;
```

Better yet, we could eliminate the indenting entirely...
```CSharp
return radius > upperLimit || radius < lowerLimit;
```

As with any rule, certain exceptions can apply.  One such exception to this rule is the use of expression bodies.

Consider the following code:
```CSharp
private bool _isNewLineBetterthanSameLine;
public bool IsNewLineBetterThanSameLine
{
    get
    {
        return _isNewLineBetterthanSameLine;
    }
}
```

While we value curly braces, we also realize the code bloat it can lend itself to.  The above is one such example.  Consider rewriting this using an expression body and eliminating the need for curly braces entirely:
```CSharp
private bool _isNewLineBetterthanSameLine;
public bool IsNewLineBetterThanSameLine => _isNewLineBetterthanSameLine;
```

Also when indenting, you should always be aware of the number of nested indents being used. Nested indents within a single function can be considered a code smell and we avoid them as much as possible.  Therefore, you should strive to never indent more than once in a single function.

Consider this hard to read, nested indenting function code:
```CSharp
public decimal GetAllOrderCosts(Order orders)
{
    var allOrderCosts = 0;
    orders.ForEach(order => {
        for (var index = 0; index <= OrderItems.length; index++)
        {
            var orderItem = OrderItems[index];
            decimal orderItemCost = 0;
            if (orderItem.IsFlatCost)
            {
                allOrderCosts += orderItem.FlatCost;
                continue;
            }
            allOrderCosts += GetCostByUnitAndQuantity(orderItem.Quantity, orderItem.UnitCost);
        };
    });
    return allOrderCosts;
}
```

The above example could be rewritten as follows:
```CSharp
public decimal GetOrderItemCost(OrderItem orderItem)
{
    if (orderItem.IsFlatCost)
    {
        return orderItem.FlatCost;
    }
    return GetCostByUnitAndQuantity(orderItem.Quantity, orderItem.UnitCost);
}

public decimal GetOrderCost(Order order)
{
    decimal orderCost = 0;
    order.Items.ForEach(orderItem =>
    {
        orderCost = GetOrderItemCost(orderItem);
    });
    return orderCost;
}

public decimal GetAllOrderCosts(List<Order> orders)
{
    var allOrderCosts = 0;
    orders.ForEach(order =>
    {
        allOrderCosts += GetOrderCost(order);
    });
    return allOrderCosts;
}
```

It should also be noted that the above example goes hand-in-hand with the core concepts previously discussed in [Functions - Do One Thing](https://github.com/AndcultureCode/AndcultureCode/blob/main/clean-code/FUNCTIONS.md#do-one-thing) and [Functions - Simple](https://github.com/AndcultureCode/AndcultureCode/blob/main/clean-code/FUNCTIONS.md#do-one-thing)

### Team Rules

We value consistency across our projects. While we will always have disagreements about
conventions, all project teams should up hold conventions until the team has agreed to make a change.

Team members should strive to make their code indistinguishable. We could go as far as to say it is a
code smell if you can tell who authored a piece of code.

That said, these documents are in no way set in stone. Proposals to change our practices are welcomed
and very much expected.
