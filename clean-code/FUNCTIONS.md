# Clean Code: Functions

Functions are the first line of organization in any program. Lets prioritize writing them well.

[Table of Contents](../CLEAN-CODE.md)

## Small!

When writing functions, strive to keep them small. As a general guideline, do your best to keep functions less than twenty lines.

By sticking to other clean code principles, this guideline will be more easily achievable. For example, by ensuring your function only does one thing, or that it doesn't have side-effects, it will naturally be more concise and readable.

## Do one thing

Functions should do one thing, and do it well. When writing a function, it should be easy to follow without multiple branching paths of execution.

When naming a function, you can sometimes identify if it's doing more than one thing by identifying what the function does in the name. For example:

```CS
public IResult<Stock> CreateStockAndUpdateFromApi(string ticker) => Do<Stock>.Try((r) =>
{
    var stock = new Stock
    {
        Ticker = ticker
    };

    var createResult = _stockRepository.Create(stock);

    if (createResult.HasErorrsOrResultIsNull())
    {
        r.AddErrors(createResult.Errors);
        return null;
    }

    var apiResponse = _stockApi.GetData(ticker);

    if (apiResponse.HasErrors)
    {
        r.AddErrors(apiResponse.Errors);
        return stock;
    }

    stock.Price = apiResponse.Data.LatestPrice;
    stock.LastClose = apiResponse.Data.Close;

    _dbContext.SaveChanges();

    return stock;
})
.Result;
```

In this case, it's clear from the name that this is doing more than one thing: it's both creating a new Stock entity, as well as updating some data based on an API. A better delineation of concerns would be as follows:

```CS
public IResult<Stock> CreateStock(string ticker) => Do<Stock>.Try((r) =>
{
    var stock = new Stock
    {
        Ticker = ticker
    };

    var createResult = _stockRepository.Create(stock);

    if (createResult.HasErorrsOrResultIsNull())
    {
        r.AddErrors(createResult.Errors);
        return null;
    }

    return stock;
})
.Result;

public IResult<bool> UpdateStockFromApi(long id) => Do<bool>.Try((r) =>
{
    var stock = _dbContext.Stocks.Where(e => e.Id == id);

    if (stock == null)
    {
        return false;
    }

    var apiResponse = _stockApi.GetData(stock.Ticker);

    if (apiResponse.HasErrors)
    {
        r.AddErrors(apiResponse.Errors);
        return false;
    }

    stock.Price = apiResponse.Data.LatestPrice;
    stock.LastClose = apiResponse.Data.Close;

    _dbContext.SaveChanges();

    return true;
})
.Result;
```

This leads to more granular functions that can be called independently from one another. It also creates more maintainable code. Each function now only has one reason to change.

Finally, beware of branching execution paths in your functions. While there are exceptions, in general you should consider it a code smell when there are boolean parameters defining different paths of logic. For example:

```CS
public IResult<decimal> GetLatestPrice(string ticker, bool updateDatabase) => Do<decimal>.Try((r) =>
{
    var stock = _dbContext.Stocks.Where(e => e.Ticker == ticker);

    if (stock == null)
    {
        return false;
    }

    var apiResponse = _stockApi.GetData(ticker);

    if (apiResponse.HasErrors)
    {
        r.AddErrors(apiResponse.Errors);
        return false;
    }

    var latestPrice = apiResponse.Data.LatestPrice;

    if (updateDatabase)
    {
        stock.Price = latestPrice;

        var updateResult = _stockRepository.Update(stock);

        if (updateResult.HasErrors)
        {
            r.AddErrors(updateResult.Errors);
            return latestPrice;
        }
    }

    return latestPrice;
})
.Result;
```

The issue with this is clear: adding an optional execution path muddies the responsibilites of the function, and gives it multiple reasons to require modification (if the process to update a stock changes, or if the process to retrieve the latest price changes).

The above example would be cleaner and easier to maintain if simply broken out into two separate functions, one to grab the latest price from the API and another to update the entity in the repository.

## One level of abstraction per function

While an uncommon consideration, remaining on a single level of abstraction can ease comprehension for readers. Avoid mixing high-level concepts with low-level details.

Consider the following example in which an API controller's `Create` action is handling both the creation of an external user as well as the local user entity.

```CS
[HttpPost]
public IActionResult Create([FromBody] UserDto dto)
{
    // -----------------------------------------------------------------------------------------
    // #region Low level abstraction - directly interacting with a REST Client/external API and handling response
    // -----------------------------------------------------------------------------------------

    var request = new RestRequest();
    request.AddJsonBody(JsonConvert.SerializeObject(dto));

    var response = _restClient.Post(request);
    if (response == null || response.ErrorException != null)
    {
        r.AddErrorAndLog(_logger, _localizer, ERROR_EXTERNAL_SERVICE_UNRESPONSIVE);
        return null;
    }

    var externalUserDto = JsonConvert.DeserializeObject<Response<ExternalUserDto>>(
        response.Content,
        new JsonSerializerSettings { NullValueHandling = NullValueHandling.Ignore }
    );

    // #endregion Low level abstraction - directly interacting with a REST Client/external API and handling response

    var user = new User
    {
        ExternalId = externalUserDto.Id,
        // ...
    };

    // -----------------------------------------------------------------------------------------
    // #region High level abstraction - persistence of a domain entity using a repository
    // -----------------------------------------------------------------------------------------

    var createResult = _createConductor.Create(user, dto.Password);
    if (createResult.HasErrors)
    {
        return InternalError<UserDto>(createResult.Errors);
    }

    // #endregion High level abstraction - persistence of a domain entity using a repository

    return Created(createResult.ResultObject);
}
```

A better way to handle this would be abstracting the external user creation into a provider class, which can know about the low-level details of hitting the external API and handling the response.

```CS
[HttpPost]
public IActionResult Create([FromBody] UserDto dto)
{
    // -----------------------------------------------------------------------------------------
    // #region High level abstraction - interaction with an external service is wrapped in a provider
    // -----------------------------------------------------------------------------------------

    var externalUserResult = _externalProvider.Create(dto);
    if (externalUserResult.HasErrors)
    {
        return InternalError<UserDto>(externalUserResult.Errors);
    }

    var externalUserDto = externalUserResult.ResultObject;

    // #endregion High level abstraction - interaction with an external service is wrapped in a provider

    var user = new User
    {
        ExternalId = externalUserDto.Id,
        // ...
    };

    // -----------------------------------------------------------------------------------------
    // #region High level abstraction - persistence of a domain entity using a repository
    // -----------------------------------------------------------------------------------------

    var createResult = _createConductor.Create(user, dto.Password);
    if (createResult.HasErrors)
    {
        return InternalError<UserDto>(createResult.Errors);
    }

    // #endregion High level abstraction - persistence of a domain entity using a repository

    return Created(createResult.ResultObject);
}
```

## Switch statements

When comparing more than two possible conditions of an expression, prefer the `switch` statement to the `if...else`. Just as `if...else` conditionals should be avoided, the `switch` should be used sparingly. They should be kept DRY and at the lowest level of abstraction possible.

```TypeScript
// Avoid if...else and leverage switch statements
if (response.status === StatusCode.Unauthorized || response.status === StatusCode.Forbidden) {
    url = Sitemap.Errors.Unauthorized;
} else if (response.status === StatusCode.NotFound) {
    url = Sitemap.Errors.NoFound;
} else if (response.status === StatusCode.ServerError) {
    url = Sitemap.Errors.ServerError;
}

// Switch statements remove redundancies of the if...else
switch (response.status) {
    case StatusCode.Unauthorized:
    case StatusCode.Forbidden: {
        url = Sitemap.Errors.Unauthorized;
        break;
    }
    case StatusCode.NotFound: {
        url = Sitemap.Errors.NoFound;
        break;
    }
    case StatusCode.ServerError: {
        url = Sitemap.Errors.ServerError;
        break;
    }
}

// Avoid both when possible
const urls = {
    [StatusCode.Unauthorized]: Sitemap.Errors.Unauthorized,
    [StatusCode.Forbidden]: Sitemap.Errors.Unauthorized,
    [StatusCode.NotFound]: Sitemap.Errors.NotFound,
    [StatusCode.ServerError]: Sitemap.Errors.ServerError
};
...
url = urls[response.status];

// Even better... align `Sitemap.Errors` with `StatusCode` values...
url = Sitemap.Errors[response.status];
```

## Use descriptive names

Creating a function name may seem easy, but take the time to make it clear and descriptive of what the function does.
Try different options and make sure that the name matches what is happening.
An additional benefit of using descriptive names is that it helps to point out when functions are breaking the single-responsibility principle and need refactored.

### Long Names

A more focused function could have a small name, but do not be afraid of writing long names.
A long descriptive name is better than a long descriptive comment.

Before:

```TypeScript
// Queries authors list and sets the values in the dropdown
public setAuthors()
```

After:

```TypeScript
// Queries authors list and sets the values in the dropdown
public populateAuthorsDropdown()
```

### Consistent Multi-Word Naming

When you consistently use the same descriptive words in function names the code is more readable, while still providing details about the functionality.

Below we consolidate the "GetApproved", "ListAuthorized" and "ReadApproved" prefixes, which are all saying the same thing, into "ListApproved".
This naming convention helps to tell a story, and may even help to reveal missing functionality or misnamed functions. For example, we are expecting to see "ListApprovedBookSeries".

| Bad                                                                       | Good                                                                     |
| ------------------------------------------------------------------------- | ------------------------------------------------------------------------ |
| GetApprovedAuthors()<br>ListAuthorizedBooks()<br>ReadApprovedPublishers() | ListApprovedAuthors()<br>ListApprovedBooks()<br>ListApprovedPublishers() |

## Function arguments

While an _ideal_ number of function arguments is zero (0), it's more often necessary to encapsulate some amount of decision making into a function.
Keeping the function small (See [Small!](#Small!)) as well as making functions flexible enought to provide clear value in a handful of situations is preferable over creating hyper-specific functions in hopes of targeting a single outcome. Remember your audience, if the reader cannot make sense of the function given the arguments, it's likely a candidate for refactoring.

When functions have four or more arguments, consider using an argument object or an options bag. The goal of this refactor is to group like arguments into concepts of their own, and/or increase clarity for the consumer.

```TypeScript
// Before
export default function useMyBookmarks(
    collectionId: number | undefined = undefined,
    searchText: string | undefined = undefined,
    colors: Array<UserBookmarkColors> | undefined = undefined,
    filterByCurrentUser: boolean = false,
    sortBy: UserBookmarkSortOption | undefined = undefined,
    skip: number | undefined = undefined,
    take: number | undefined = undefined,
    loadSections: boolean = true,
    loadPublications: boolean = true,
    loadCollections: boolean = true,
    loadUsers: boolean = true
) { ... }

// After
export interface UserBookmarkIndexParams {
    collectionId?: number,
    searchText?: string,
    sortBy?: userBookmarkSortOption,
    skip?: number,
    take?: number
}

export interface UserBookmarkIncludeProperties {
    sections: boolean,
    publications: boolean,
    collections: boolean,
    users: boolean
}

export default function useMyBookmarks(
    apiParams: UserBookmarkIndexParams,
    includeProperties: UserBookmarkIncludeProperties,
) { ... }

```

When naming arguments, choose names that create a verb-noun pair between the function and the argument. The function should be a name reflecting a verb, and the arguments a noun.

```TypeScript
function writeField(name: string) { ... }

function assertExpectedEqualsActual(expected: any, actual: any) { ... }
```

## Have no side effects

A [side effect](<https://en.wikipedia.org/wiki/Side_effect_(computer_science)>) is when a function modifies a value outside its local scope or environment.
It is doing more than the function's stated purpose, which can lead to strange couplings and hard to debug order dependencies.
A helpful goal is to write [pure functions](https://en.wikipedia.org/wiki/Pure_function), which by definition, will result in functions without side effects.
If a side effect is unavoidable, it is good to name the function so that it's impact and intention is understood by the reader.

_Before_

```TypeScript
function checkUserPermission(
    username: string,
    permissions: string[],
): boolean {
    const user = UserService.getUser(username);

    if (user == null) {
        return false;
    }

    const userPermissions = PermissionService.getPermissions(user);

    if (CoreUtils.hasValues(userPermissions) &&
        permissions.some((permission) =>
            userPermissions.includes(permission))) {

        // Side-effect
        GlobalState.initialize();

        return true;
    }

    return false;
}
```

_After_

```TypeScript
function checkUserPermissionsAndInitializeGlobalState(
    username: string,
    permissions: string[],
): boolean {
    const user = UserService.getUser(username);

    if (user == null) {
        return false;
    }

    const userPermissions = PermissionService.getPermissions(user);

    if (CoreUtils.hasValues(userPermissions) &&
        permissions.some((permission) =>
            userPermissions.includes(permission))) {

        // Side-effect
        GlobalState.initialize();

        return true;
    }

    return false;
}
```

## Command Query Separation

**Do** something...

```TypeScript
// local-storage-utils.ts

/**
 * Cache key value pair to local storage as strings
 */
function set<T>(key: string, value: T): void {
    localStorage.setItem(key, JSON.stringify(value));
}
```

Or **Answer** something...

```TypeScript
// local-storage-utils.ts

/**
 * Returns whether or not the provided local storage key exists
 */
public has(key: string): boolean {
    return localStorage.getItem(key) != null;
}
```

_NOT_ both...

```TypeScript
// local-storage-utils.ts

/**
 * Cache key value pair to local storage as strings.
 * Returns true if key successfully set or false if not set
 */
function set<T>(key: string, value: T): boolean {
    localStorage.setItem(key, JSON.stringify(value));
    return has(key);
}
```

## Prefer Exceptions to Returning Error Codes

When there is an exception, **do not** catch it and return an error code.
This causes a problem for the caller, because they must immediately deal with the error even if they are not the appropriate place to handle it. This approach is largely antiquated.

```CSharp
// Do not return error codes

public CreateResponse CreateFile(File file) {
    try {
        FunctionThatThrowsAnException(file);
        return CreateResponse.Success;
    } catch (AccessException ex) {
        return CreateResponse.AccessIssues;
    } catch (Exception ex) {
        return CreateResponse.Exception;
    }
}
```

It is **unnecessary** to split the try and logic code into separate functions.
While this does separate the code into two different things (creating file and handling exceptions), it is not necessary when we use shared error handling.

```CSharp
// Do not split catching and functionality

public File CreateEmptyFile() {
    return CreateFile(new File(...))
}

public File CreateFile(File file) {
    try {
        return FunctionThatThrowsAnException(file);
    } catch (Exception ex) {
        LogException(ex);
        return null;
    }
}
```

We should also use shared "catching" code (such as `Do/Try`) that handles the exceptions consistently.
In this case any exceptions from `CreateEmptyFile()` are caught by `PopulateEmptyFile()`.

```CSharp
// Used shared catching code

public IResult<File> PopulateEmptyFile(string content) => Do<File>.Try((r) =>
    var file = CreateEmptyFile();
    // populate functionality
    return file;
}).Result;
```

## Structured Programming

Structured programming is defined by functions having a single entry and a single exit point. This principle limits each function to a single `return` statement. Writing functions in such a way that only a single return is needed generally benefits a large function. However, when functions are kept small, short circuiting functions with `return`, `break` and `continue` can often be more expressive.

Below is a common case for breaking Structured Programming by short circuiting the function. Alternatively, an `alignmentStyle` variable could be reassigned based on the `alignments.includes(#style)`, returning a single reassigned variable in an effort to maintain a structured format.

```TypeScript

public getAlignmentStyle(styles: string[]): TableCellAlignmentStyle {
    const alignments: string[] = styles.filter((x) =>
        x.match("^align") ? x : null;
    );
    if (alignments.includes("align-left")) {
        return TableCellAlignmentStyle.Left;
    }
    if (alignments.includes("align-right")) {
        return TableCellAlignmentStyle.Right;
    }
    if (alignments.includes("align-center")) {
        return TableCellAlignmentStyle.Center;
    }
    if (alignments.includes("align-justify")) {
        return TableCellAlignmentStyle.Justify;
    }
    return TableCellAlignmentStyle.None;
    }
```

## How Do You Write Functions Like This?

No code should ever be considered complete or unchangeable. The goal is to make code easy to digest and maintain for future readers (ourselves included).

When creating new functions, getting it "working" is just the first step. Before committing a new changeset, engineers should take time, even if only ten minutes, to review their work and apply concepts outlined in this and other clean code documentation while it is fresh in your mind.

Pair programming with another engineer on your team is even better! Ideally getting a review from engineer(s) that aren't on your project team.

In the event you do not have time to perform an initial refactoring, an issue/task should be created in the respective project management tool and left in a comment on the file(s)/functions/etc...

No matter what, you _must_ take the time to re-read your changeset top-to-bottom and ensure there aren't any glaring issues before committing it to the codebase. If you feel there isn't enough time for even this step, please discuss how you can get more time to prioritize this important step with your managers.
