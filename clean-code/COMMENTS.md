# Clean Code: Comments

[Table of Contents](../CLEAN-CODE.md)

## Comments Do Not Make Up for Bad Code

Think long and hard before adding a comment. Using the principles in this book as a guide, attempt
to express yourself in code whenever possible.

Comments suggesting a refactor later should be _rare_ and prioritized to be completed shortly
after their creation.

## Explain Yourself in Code

Code should be self-documenting whenever possible. Breaking out complex logical expressions into
variables or functions can help readability and replace the need for a comment at all.

**Bad**

```tsx
{
    return (
        <React.Fragment>
            {// Only show the admin actions if user has admin, publisher or author roles
            (user.hasRole(RoleType.SystemAdmin) ||
                user.hasRole(RoleType.Publisher) ||
                user.hasRole(RoleType.Author)) && (
                <AdminContextMenu {...props} />
            )}
        </React.Fragment>
    );
}
```

**Better**

Break out a variable to reduce nesting and capture a higher-level concept.

```tsx
{
    const hasEditingAccess =
        user.hasRole(RoleType.SystemAdmin) ||
        user.hasRole(RoleType.Publisher) ||
        user.hasRole(RoleType.Author);

    return (
        <React.Fragment>
            {hasEditingAccess && <AdminContextMenu {...props} />}
        </React.Fragment>
    );
}
```

**Ideal**

Combine view model methods to capture this higher-level concept and reduce duplication.

```ts
// user-record.ts
public hasAdminRole(): boolean {
    return this.hasRole(RoleType.SystemAdmin) ||
        this.hasRole(RoleType.Publisher) ||
        this.hasRole(RoleType.Author);
}
```

```tsx
// component-detail.tsx
{
    return (
        <React.Fragment>
            {user.hasAdminRole() && <AdminContextMenu {...props} />}
        </React.Fragment>
    );
}
```

## Good Comments

### Informative Comments

Comments can be used as additional information for code that is difficult to decipher, even in the
best circumstances. For instance, even well named regex patterns can easily lose context.

```CSharp
// Replace spaces before or after inline elements (i.e /inline>\s+< or >\s+<inline)
XElement.Parse(
    Regex.Replace(bodyAsXml, @"((?<=\/inline)>\s+<)|(>\s+<(?=inline))", "><span>&#xA0;</span><")
);
```

### Explanation of intent

Providing context to the decision to write code in a certain way is an acceptable form of
commenting. Interesting decisions can be reconciled through a short comment explaining why
code was written in a certain way. However, beware of comments that can become outdated or incorrect
over time as implementations change.

```CSharp
// In the event of a retry, check if this Section is already created. Ignore query filters
// to find unpublished since the publication is still importing.
IResult<bool> importResult = null;
var preexistingSectionReadResult = _sectionReadConductor.FindAll(
    filter: e =>
        e.PublicationId == publicationId &&
        e.ExternalId == section.ExternalId &&
        e.Label == section.Label &&
        e.RootSectionId == section.RootSectionId,
    ignoreQueryFilters: true
);
```

### Clarification

Similar to explaining intent, sometimes it's helpful to include a translation to obscurities
for the sake of making code more readable. In most cases, it's advised to refactor in a way that
removes the need for a comment. For example, when code is written with syntax the team would
traditionally shy away from for the sake of a purpose, clarifying that purpose is worth the comment.

```CSharp
// Use LINQ syntax in place of expression syntax for optimal readability and query performance
var currentSections =
    from section in latestSectionsResult.ResultObject
    join bookmark in userBookmarks on section.ExternalId equalsbookmark.ExternalId
    join publication in latestPublications
        on new
        {
            Code = bookmark.PublicationCode,
            Edition = bookmark.PublicationEdition,
            Id = section.PublicationId
        }
        equals new { publication.Code, publication.Edition, publication.Id }
    select section;
```

### Warning of Consequences

Comments adding cautionary statements to other developers call the safety of the code itself.
In the below case, consider removing the test completely or skipping it. You can always get it
back.

```java
// Don't run unless you have some time to kill
    public void _testWithReallyBigFile()
    {
        writeLInesToFile(1000000);

        response.setBody(testFile);
        response.readyToSend(this);
        String responseString = output.toString();
        assertSubString("Content-Length: 1000000", responseString);
        assertTrue(bytesSent > 1000000);
    }
```

### TODO Comments

TODO Comments should not be left in the codebase without a paired PBI/Issue Number. Missing PBI's
should be called out in code review. Please follow the below template or similar.

```typescript
public includeSectionLabel(): boolean {
    // TODO: Pending resolution of [NFPA-3522](https://app.clickup.com/t/2219993/NFPA-3522)
    // Long term decision on whether labels will be displayed
    return this.type === PublicationTypes.NEC;
}
```

### Amplification

Comments used to amplify the importance of code present a good use case for writing a unit test to
enforce something seemingly inconsequential.
However, within automated tests we do prefer important setup be called out with a brief explanation
why that line is so important.

```CSharp
var annexes = new List<Annex>
    {
        // This is an important setup step, including an Article with a reference in its body
        Create<Annex>(
            (e) => e.Body = Build<XElement>(XElementFactory.WITH_EXTERNAL_ID).ToString(),
            (e) => e.PublicationId = publication.Id
        ),
        Create<Annex>((e) => e.PublicationId = publication.Id),
    };
```

### Doc Comments in Public API's

Javadocs and Summary comments can be used to generate documentation through OpenAPI documentation
tools like Swagger. For public API's this is useful to the consumer and is easy to maintain by the
development team.

```CSharp
 #region HTTP Post

    /// <summary>
    /// Creates a new UserBookmark record
    /// </summary>
    /// <remarks>
    /// Sample request:
    ///
    ///     POST /UserBookmarkDto
    ///     {
    ///         "createdOnPublicationId": 1,
    ///         "description": "<b>New UserBookmark</b>",
    ///         "descriptionAsPlainText": "New UserBookmark",
    ///         "externalId": "ID000700000002",
    ///         "userId": 1,
    ///     }
    ///
    /// </remarks>
    /// <param name="userBookmarkDto">UserBookmark to be created</param>
    /// <returns>Created UserBookmark</returns>
    /// <response code="201">UserBookmark successfully created</response>
    /// <response code="400">Provided UserBookmark is invalid</response>
    /// <response code="401">User is not authenticated</response>
    /// <response code="403">User is not authorized to perform this action</response>
    /// <response code="404">Resource or related resource not found</response>
    /// <response code="500">Errors were encountered creating data</response>
    [AclAuthorize(AclStrings.USERBOOKMARKS_CREATE)]
    [HttpPost]
    [ProducesResponseType(201, Type = typeof(UserBookmarkDto))]
    [ProducesResponseType(400, Type = typeof(UserBookmarkDto))]
    [ProducesResponseType(401, Type = typeof(UserBookmarkDto))]
    [ProducesResponseType(403, Type = typeof(UserBookmarkDto))]
    [ProducesResponseType(404, Type = typeof(UserBookmarkDto))]
    [ProducesResponseType(500, Type = typeof(UserBookmarkDto))]
    public IActionResult Create([FromBody] UserBookmarkDto userBookmarkDto)

```

## Bad Comments

### Mumbling

Don't add comments just because you feel that you have to.
This can result in rushed comments that don't make sense, or comments that require you to dig into code to understand the comment.

### Redundant Comments

Comments should not repeat what is in the function name, nor what is evident in small chunks of code.
Reading the comment shouldn't take longer than reading and understanding the code.

### Misleading Comments

Make sure that comments are precisely correct and are not misleading.
When making changes, check that comments are still correct.
If you encounter comments that are not correct, update them no matter who the author was.
In addition, do not place comments after code being commented on; it should be before the code.

### Mandated Comments

It is silly to require that every function and variable have comments.

However, our team is fine with requiring comments for:

-   Code Region names or descriptions
-   Arrange, Act and Assert comments within tests

### Journal Comments

Do not add a journal of updates to functions, files or modules.
One exception would be in something outside of source control, such as stored procedures, but we should avoid SPs when possible.

```
2021-02-11 rcrusoe Created bad journal comment
2021-02-15 rcrusoe Unnecessary comment
2021-02-16 rcrusoe Another unnecessary comment
```

### Scary Noise

Unnecessary comments are bad enough, but if copying and pasting, ensure that comments are updated (if still helpful).
You do not want to have a comment that does not match the element (it wastes a lot of brain power).

### Position Markers

Use regions within code, not comments with repeated characters.

```CSharp
// Bad Position Marker --------------------------
```

```CSharp
#region Good Position Marker
```

In some cases a combined solution could be agreed upon by the team, such as this position/region marker in TypeScript.

```TypeScript
// -------------------------------------------------------------
// #region Private Properties
// -------------------------------------------------------------

private _groupToOpen:       string;
private _sendIntervalDelay: number;

// #endregion Private Properties
```

Region snippets for VS Code are saved under the prefixes "[ts-region](https://github.com/AndcultureCode/AndcultureCode/blob/d0f70f16ef98edfe5069f6c52c56d1e84e2426e5/.vscode/typescript.code-snippets#L18-L28)" and "[cs-region](https://github.com/AndcultureCode/AndcultureCode/blob/d0f70f16ef98edfe5069f6c52c56d1e84e2426e5/.vscode/csharp.code-snippets#L8-L12)".

### Commented-Out Code

Do not leave commented-out code unless there is a very good reason (preferably with a note as to why it still exists).
Because we work in source control, it is easy to restore code if necessary.
Once it is in source control, others are reluctant to remove it since there _surely_ must have been a good reason for it to be added.

### HTML Comments

If comments are going to be extracted by an external tool, it should be the responsibility of that tool to format the content.

However, we agreed to allow markup in comments that will be used for documentation, such as Swagger.

### Too Much Information

Keep your comments brief.
Do not include historical or irrelevant details within the comments.

Also, do not include information that is not relevant to the current context.
Additional content makes it a higher likelihood that the comments and the code will diverge.
