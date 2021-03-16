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

https://github.com/AndcultureCode/AndcultureCode/issues/68

## Horizontal Formatting

### General

Strive to maintain a 100 character width maximum per line (shorter is typically better). Ideally,
this would be enforced by an auto-formatter or linting tools, but lines that are hard to read due to
length can also be called out in code reviews.

### Alignment

https://github.com/AndcultureCode/AndcultureCode/issues/70

### Indentation

https://github.com/AndcultureCode/AndcultureCode/issues/71

### Team Rules

We value consistency across our projects. While we will always have disagreements about
conventions, all project teams should up hold conventions until the team has agreed to make a change.

Team members should strive to make their code indistinguishable. We could go as far as to say it is a
code smell if you can tell who authored a piece of code.

That said, these documents are in no way set in stone. Proposals to change our practices are welcomed
and very much expected.
