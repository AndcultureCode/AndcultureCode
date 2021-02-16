# Clean Code: Comments

[Table of Contents](../CLEAN-CODE.md)

## Comments Do Not Make Up for Bad Code

Think long and hard before adding a comment. Using the principles in this book as a guide, attempt
to express yourself in code whenever possible.

Comments suggesting a refactor later should be _rare_ and prioritized to be completed shortly
after their creation.

## Explain Yourself in Code

## Good Comments

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

```
// Bad Position Marker --------------------------
```

```
#region Good Position Marker
```

In some cases a combined solution could be agreed upon by the team, such as this for Typescript in Edio:

```
// -------------------------------------------------------------
// #region Private Properties
// -------------------------------------------------------------

private _groupToOpen:         string;
private _localStorageActions: LocalStorageActions;
private _sendIntervalDelay:   number;

// #endregion Private Properties
```

### Commented-Out Code

Do not leave commented-out code unless there is a very good reason (preferably with a note as to why it still exists).
Because we work in source control, it is easy to restore code if necessary.
Once it is in source control, others are reluctant to remove it since there _surely_ must have been a good reason for it to be added.

### HTML Comments

If comments are going to be extracted by an external tool, it should be the responsibility of that tool to format the content.

However, we agreed to allow markup in comments that will be used for documentation, such as Swagger.

### Too Much Information

Do not include historical or irrelevant details within the comments.
Keep your comments brief.
