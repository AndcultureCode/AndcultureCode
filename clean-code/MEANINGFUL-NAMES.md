# Clean Code: Meaningful Names

Simple rules for creating good names.

[Table of Contents](../CLEAN-CODE.md)

## Meaningful Names

### Use Intention-Revealing Names

The name of a thing should answer what it does, how it's used, and why it exists. Simply by reading
the name you should have some sense of the intention.

For example, the following code doesn't indicate the intention:

```CS
public bool Compare(List<Product> list, float ct)
{
    var c = 19.99f; // Products below this cost should not contribute to the total.
    var tc = 0;

    foreach (var l in list)
    {
        var more = l.Cost > c;

        if (!more)
        {
            continue;
        }

        tc += l.Cost;
    }

    return tc > ct;
}
```

Reading the code above gives you very little information about what exactly it's supposed to be
doing. Simply by renaming variables, methods, etc., you can add significant clarity to your code.
For example, the above could be rewritten as such:

```CS
public bool IsProductsTotalCostAboveThreshold(List<Product> products, float costThreshold)
{
    var minimumCost = 19.99f; // Products below this cost should not contribute to the total.
    var totalCost = 0;

    foreach (var product in products)
    {
        var costBelowMinimum = product.Cost <= minimumCost;

        if (!costBelowMinimum)
        {
            continue;
        }

        totalCost += product.Cost;
    }

    return totalCost > costThreshold;
}
```

Now the intention of the method is clear (although contrived). No comments needed to be added, while
it could be refactored to be less verbose, by simply renaming the method and related variables, the
code is significantly more digestible.

### Make Meaningful Distinctions

#### Avoid Noise Words

Noise words are words that don't provide additional information, aren't known patterns used in
software, and are often added to differentiate similar names. Examples of noise words:

1.  Amount
1.  Data
1.  Info
1.  Object
1.  The
1.  (Any name that includes the item type)

Instead of using words like these, add context to the names. A single name may not be incorrect, but
when existing alongside similar names, it becomes a problem.

| Bad                                            | Good                                                         |
| ---------------------------------------------- | ------------------------------------------------------------ |
| Product<br>ProductData<br>ProductInfo          | ProductDetail<br>ProductListItem<br>HomePageProduct          |
| GetActiveAccount()<br>GetActiveAccountRecord() | GetActiveAccountFullDetails()<br>GetActiveAccountForHeader() |

#### Names Should have Meaningful Distinctions

Being able to differentiate classes, methods or variables isn't enough, they should have meaningful
distinctions. This means that we shouldn't just iterate a number over items. The names should show
why they are different than the other similar objects. In tests the distinction may be whether the
result is expected or not. Keep in mind that if you are creating multiple records that do not need
to be read, they could be unnamed and initiated within an array.

| Bad                                                                                                                                                | Good                                                                                                                                                                          |
| -------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| let user1 = null;<br>let user2 = null;<br>let user3 = null;                                                                                        | let studentUser = null;<br>let parentUser = null;<br>let teacherUser = null;                                                                                                  |
| let product1 = null;<br>let product2 = null;                                                                                                       | let expectedProduct = null;<br>let unexpectedProduct = null;                                                                                                                  |
| let expectedWidget = create(1);<br>let unexpectedWidget2 = create(2);<br>let unexpectedWidget3 = create(3);<br> let unexpectedWidget4 = create(4); | let expectedWidget = create(1);<br>const options = [ expectedWidget, create(2), create(3), create(4) ];<br>/\* assert on results length rather than not having unexpected \*/ |

### Don't Add Gratuitous Context

When a variable name can be inferred from the context of the class, interface or other actor you're in, prefixing the variable with the additional context adds no value. Extra context should only be added for actors that differ from the domain/context.

For example, in an API controller that performs read operations on a `Chapter` entity, your first instinct might be to name the repository conductor prefixed with "chapter":

```CS
[FormatFilter]
[ApiRoute("chapters")]
public class ChaptersController : ApiController
{
    #region Private Properties

    // The "chapter" prefix here is redundant due to the context of being in the `ChaptersController`
    private readonly IRepositoryReadConductor<Chapter> _chapterReadConductor;
    private readonly ILogger<ChaptersController> _logger;
    private readonly IMapper _mapper;

    #endregion Private Properties

    #region Constructor

    public ChaptersController(
        /* The "chapter" prefix here is redundant due to the context of being in the `ChaptersController` */
        IRepositoryReadConductor<Chapter> chapterReadConductor,
        IStringLocalizer localizer,
        ILogger<ChaptersController> logger,
        IMapper mapper
    ) : base(localizer)
    {
        // The "chapter" prefix here is redundant due to the context of being in the `ChaptersController`
        _chapterReadConductor = chapterReadConductor;
        _logger = logger;
        _mapper = mapper;
    }

    #endregion Constructor

    // Get, Index, etc. controller methods here
}
```

However, in this context, we know that the primary entity we are working with is a `Chapter`. Prefixing the conductor with "chapter" is superfluous and can be removed. This would be more concise and provide the same amount of information:

```CS
[FormatFilter]
[ApiRoute("chapters")]
public class ChaptersController : ApiController
{
    #region Private Properties

    private readonly ILogger<ChaptersController> _logger;
    private readonly IMapper _mapper;
    // Since the entity for this conductor is our primary entity in this controller, we don't need to prefix it with "chapter"
    private readonly IRepositoryReadConductor<Chapter> _readConductor;

    #endregion Private Properties

    #region Constructor

    public ChaptersController(
        IStringLocalizer localizer,
        ILogger<ChaptersController> logger,
        IMapper mapper,
        /* Since the entity for this conductor is our primary entity in this controller, we don't need to prefix it with "chapter" */
        IRepositoryReadConductor<Chapter> readConductor,
    ) : base(localizer)
    {
        _logger = logger;
        _mapper = mapper;
        // Since the entity for this conductor is our primary entity in this controller, we don't need to prefix it with "chapter"
        _readConductor = readConductor;
    }

    #endregion Constructor

    // Get, Index, etc. controller methods here
}
```

### Add Meaningful Context

Of course, when there are additional actors that do not directly relate to the domain entity, it then becomes appropriate to make the distinction:

```CS
[FormatFilter]
[ApiRoute("chapters")]
public class ChaptersController : ApiController
{
    #region Private Properties

    private readonly ILogger<ChaptersController> _logger;
    private readonly IMapper _mapper;
    // Since this entity now differs from our primary entity `Chapter`, we will make the distinction in the name.
    private readonly IRepositoryReadConductor<Publication> _publicationReadConductor;
    private readonly IRepositoryReadConductor<Chapter> _readConductor;

    #endregion Private Properties

    #region Constructor

    public ChaptersController(
        IStringLocalizer localizer,
        ILogger<ChaptersController> logger,
        IMapper mapper,
        /* Since this entity now differs from our primary entity `Chapter`, we will make the distinction in the name. */
        IRepositoryReadConductor<Publication> publicationReadConductor,
        IRepositoryReadConductor<Chapter> readConductor,
    ) : base(localizer)
    {
        _logger = logger;
        _mapper = mapper;
        // Since this entity now differs from our primary entity `Chapter`, we will make the distinction in the name.
        _publicationReadConductor = publicationReadConductor;
        _readConductor = readConductor;
    }

    #endregion Constructor

    // Get, Index, etc. controller methods here
}
```

### Avoid Encodings

While adding encoded details to a name such as type and scope were needed in the past, the combination of C# being a strongly typed language as well as having powerful IDE support has made this an obsolete requirement. Besides it no longer being needed for languages such as C#, there are a few other reasons that encoding names should be avoided. First, it adds an unneeded burden on the reader of the code. Secondly, adding endoded information to a name introduces the potential to mislead a reader if a name or type are ever refactored in a way that they no longer are true or align.

Here are some examples:

| Bad Names              | Good Names          |
| ---------------------- | ------------------- |
| string[] nameArray;    | string[] names;     |
| List<string> nameList; | List<string> names; |
| int intAge;            | int age;            |
| string addressString;  | string address;     |

### Class Names

Class names should be nouns, not verbs. In addition, keep in mind the prior section on avoiding
noise words (Info, Data, etc). For example:

| Class Name           |
| -------------------- |
| CourseCloneConductor |
| UrlsPlugin           |
| Product              |

### Method Names

Method names should be verbs or verb phrases indicating the work they are doing. Remember, you don't
need to add unnecessary context when naming a method -- if you have a class called `Product` and a
method to initialize some properties, you may be tempted to call the method `InitializeProduct`.
However, a better name might simply be `Initialize` because based on the context of the class in which
the method exists, you are already indicating that the thing being initialized is a product. For example:

| Bad Method Name                    | Good Method Name             |
| ---------------------------------- | ---------------------------- |
| CourseCloneConductor.CloneCourse() | CourseCloneConductor.Clone() |
| UrlsPlugin.InitializeUrls()        | UrlsPlugin.Initialize()      |
| Product.DisposeProduct()           | Product.Dispose()            |

### Use One Word Per Concept

There are many ways to say the same thing! Avoid using all of them, and instead use one and only one throughout the codebase.

By this, we mean to say that each unique abstract concept should use the same word to describe itself, to reduce confusion and uncertainty when reading/writing code.

Below are two example interfaces... one follows this suggestion and the other doesn't. It's clear which is more consistent and less confusing.

| Bad                          | Good                       |
| ---------------------------- | -------------------------- |
| int GetUserId()              | int GetUserId()            |
| string ReadUserName          | string GetUserName         |
| UserAddress FetchUserAddress | UserAddress GetUserAddress |

In the Bad example, what would the difference between `Read`. `Get`, and `Fetch` be? Likely nothing, and one of these terms would suffice to describe the same concept.

Additionally, avoid using different names to describe similar patterns. In our codebase it is likely you'll see the `Conductor` pattern being used, but in other code bases you may see `Service` or `Manager`.

The terminology used to describe these patterns should be as consistent as possible. Consult team leads/senior team members for the "correct" pattern name you should use if you're unsure!
