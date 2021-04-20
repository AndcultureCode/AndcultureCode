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
and `scratchDirectory` are data structures (with accessors) or objects.
If the classes are data structures with no methods, and are written like the following, we would
know that the law is not being violated:

```csharp
var outputDirectory = context.Options.ScratchDirectory.AbsolutePath;
```

### Hybrids

The confusion of whether a class should be an object or a data structure leads to hybrid
structures that are half object and half data structure.
These classes have methods that do significant things, but also include public variables.
Don't create hybrid classes.

### Hiding Structure

To avoid issues with train wrecks, and comply with the Law of Demeter, we should create methods
that hide the structure.
For the example code above, why do we want the scratch directory?
If it is to create an output stream for a file, why not do this instead:

```csharp
var outputStream = context.CreateScratchFileStream(classFileName);
```

This seems like a reasonable method to include, while hiding internal functionality and obeying
the Law of Demeter.
