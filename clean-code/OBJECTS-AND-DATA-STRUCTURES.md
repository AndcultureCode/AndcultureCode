# Clean Code: Objects and Data Structures

Simple rules for writing Objects and Data Structures.

[Table of Contents](../CLEAN-CODE.md)

## Data Structure/Object Anti-Symmetry

Data Structures are the inverse of Objects.

An Object hides its data behind a set of functions, acting as abstractions and actors of that data.

A Data Structure shares it's data pubicly, and has no functions inherent to the data structure to act upon that data.

### Data Structures

In our projects Data Structures often take the form of Models, Entities, or DTOs. An example of one of these would be the following:

```CS
public class User
{
    public string FirstName { get; set; }
    public string LastName  { get; set; }
    public string Email     { get; set; }
    public string Password  { get; set; }
}
```

In Typescript we often build intefaces to shape our data structures:

```ts
export interface User {
    FirstName: string;
    LastName:  string;
    Email:     string;
    Password:  string;
}
```

### Objects

In our projects we often take more functional approaches to business logic implementation. However, an example of the asymmetry of Objects to Data Structures can be viewed in the way we often use Immutable classes in our front end.

An example of how we'd often use Objects can be seen in our use of Immutable, and our Record pattern ont he front end.

```ts
export class UserRecord {
    constructor(params?: Partial<User>) {
        if (params == null) {
            params = Object.assign({}, defaultValues);
        }

        super(params);
    }

    public displayName(): string {
        return `${this.firstName} ${this.lastName}`;
    }

    public hasFirstName(): boolean {
        return StringUtils.hasValue(this.firstName);
    }

    public hasLastName(): boolean {
        return StringUtils.hasValue(this.lastName);
    }
}
```

A C# equivalent of this might look something like:

```cs
public class UserRecord
{
    private readonly string _firstName;
    private readonly string _lastName;
    private readonly string _email;
    private readonly string _password;

    public UserRecord(User user)
    {
        _firstName = user.FirstName;
        _lastName  = user.LastName;
        _email     = user.Email;
        _password  = user.Password;
    }

    public string getDisplayName() => $"{_firstName} {_lastName}";

    public bool hasFirstName() => !string.IsNullOrEmpty(_firstName);

    public bool hasLastName() => !string.IsNullOrEmpty(_lastName);
}
```

When writing code, be careful not to mix these two concepts. Keeps your Data Structures devoid of business logic, and do not expose the underlying data of your Objects.
