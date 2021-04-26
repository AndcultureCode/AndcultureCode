## The Law of Demeter

The Law of Demeter is a design guideline stating that a module should not know about the inner
workings of _objects_ that it manipulates.
This allows invoking methods of objects created by it or objects passed to it, but does not allow
invoking methods of objects returned by these methods.

### Train Wrecks

Chained method calls are referred to as train wrecks since they look like a line of
coupled train cars:

```csharp
var outputDirectory = context.GetOptions().GetScratchDirectory().GetAbsolutePath();
```

This would be better split up as:

```csharp
var options = context.GetOptions();
var scratchDirectory = options.GetScratchDirectory();
var outputDirectory = scratchDirectory.GetAbsolutePath();
```

But whether this defies the Law of Demeter or not depends if the classes for `context`, `options`
and `scratchDirectory` are data structures or objects.
C# muddies the water with the ability to have advanced functionality within `get`/`set` object
accessors.
If you know that all of the classes are DTOs, then the following structure could be used
(though it isn't ideal):

```csharp
var outputDirectory = context.Options.ScratchDirectory.AbsolutePath;
```

### Hybrids

The confusion of whether a class should be an object or a data structure leads to hybrid
structures that are half object and half data structure.
These classes have methods that do significant things, but also include public variables.
Don't create hybrid classes.

### Hiding Structure

To avoid issues with train wrecks, comply with the Law of Demeter, and avoid confusion with
whether a class is a DTO or object, we should create methods that hide the structure.
For the example code above, why do we want the scratch directory?
If it is to create an output stream for a file, why not do this instead:

```csharp
var outputStream = context.CreateScratchFileStream(classFileName);
```

This seems like a reasonable method to include, while hiding internal functionality and obeying
the Law of Demeter.
