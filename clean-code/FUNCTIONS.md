# Clean Code: Functions

Functions are the first line of organization in any program. Lets prioritize writing them well.

[Table of Contents](../CLEAN-CODE.md)

## Small!

-   [TODO: Issue #33 - Book Club: Document our use of "Clean Code: Functions - Small!"](https://github.com/AndcultureCode/AndcultureCode/issues/33)
-   book = strive for max of 20, us = we will use as general guideline, but not hard requirement
-   by following other principles, helps get toward small

## Do one thing

-   [TODO: Issue #34 - Book Club: Document our use of "Clean Code: Functions - Do one thing"](https://github.com/AndcultureCode/AndcultureCode/issues/34)
-   naming concept - follow what function does
-   code smell if you are adding “and”, “or”... to your names
-   another code smell, boolean parameters as immediate branches of logic

## One level of abstraction per function

-   [TODO: Issue #35 - Book Club: Document our use of "Clean Code: Functions - One level of abstraction per function"](https://github.com/AndcultureCode/AndcultureCode/issues/35)
-   agree and perhaps mention it in documentation with example being it isn’t something many are thinking about

## Switch statements

-   [TODO: Issue #36 - Book Club: Document our use of "Clean Code: Functions - Switch Statements"](https://github.com/AndcultureCode/AndcultureCode/issues/36)
-   if necessary, should live at lowest level of abstraction that is “appropriate”
-   example: reducers, events, “command-pattern”
-   avoid business rules
-   if, if else, if else, else (same goes here) - worse form
-   try to avoid by using other rules

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

A [side effect](<https://en.wikipedia.org/wiki/Side_effect_(computer_science)>) is when a function modifies a value outside its local scope or environment. It is doing more than the function's stated purpose, which can lead to strange couplings and hard to debug order dependencies. If a side effect is unavoidable, it is good to name the function so that it's impact and intention is understood by the reader. Another helpful goal is to write [pure functions](https://en.wikipedia.org/wiki/Pure_function), which by definition, will result in functions without side effects.

_Before_

```TypeScript
function checkUserPermission(
    username: string,
    permissions: string[],
): boolean {
    const user = UserService.getUser(username);

    if (user != null) {
        const userPermissions = PermissionService.getPermissions(user);

        if (CoreUtils.hasValues(userPermissions) &&
            permissions.some((permission) =>
                userPermissions.includes(permission))) {

            // Side-effect
            GlobalState.initialize();

            return true;
        }
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

    if (user != null) {
        const userPermissions = PermissionService.getPermissions(user);

        if (CoreUtils.hasValues(userPermissions) &&
            permissions.some((permission) =>
                userPermissions.includes(permission))) {

            // Side-effect
            GlobalState.initialize();

            return true;
        }
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

## Prefer exceptions to returning error codes

-   [TODO: Issue #41 - Book Club: Document our use of "Clean Code: Functions - Prefer exceptions to returning error codes"](https://github.com/AndcultureCode/AndcultureCode/issues/41)
-   largely antiquated
-   extracting the “try” portion is too far, but understand shared “catches”

## Structured Programming

-   [TODO: Issue #42 - Book Club: Document our use of "Clean Code: Functions - Structured Programming"](https://github.com/AndcultureCode/AndcultureCode/issues/42)
-   if functions are small, breaks/continues/next are “okay” - isn’t hard an fast

## How Do You Write Functions Like This?

No code should ever be considered complete or unchangeable. The goal is to make code easy to digest and maintain for future readers (ourselves included).

When creating new functions, getting it "working" is just the first step. Before committing a new changeset, engineers should take time, even if only ten minutes, to review their work and apply concepts outlined in this and other clean code documentation while it is fresh in your mind.

Pair programming with another engineer on your team is even better! Ideally getting a review from engineer(s) that aren't on your project team.

In the event you do not have time to perform an initial refactoring, an issue/task should be created in the respective project management tool and left in a comment on the file(s)/functions/etc...

No matter what, you _must_ take the time to re-read your changeset top-to-bottom and ensure there aren't any glaring issues before committing it to the codebase. If you feel there isn't enough time for even this step, please discuss how you can get more time to prioritize this important step with your managers.
