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

## Data Transfer Objects

A Data Transfer Object (commonly abbreviated DTO) is a specific type of data structure used for
transferring data between actors/systems, such as an entity from the database to the consumer of an API.

The same rules of thumb for regular data structures apply - don't add any functions or business logic.

Below is an example DTO and its corresponding entity. While DTOs are commonly 1:1 mapped with their
database-backed entity, they do not need to be. This is one of the arguments for DTOs - it provides
a layer to allow for data to be hidden/transformed from its original source if needed.

```cs
public class User : Auditable
{
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public string PhoneNumber { get; set; }
    public string UserName { get; set; }
}
```

```cs
public class UserDto : AuditableDto
{
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public string PhoneNumber { get; set; }
    public string UserName { get; set; }
}
```
