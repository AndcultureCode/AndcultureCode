# Clean Code: Boundaries

Best practices for keeping the boundaries in our code clean.

[Table of Contents](../CLEAN-CODE.md)

### Using Third-Party Code

https://github.com/AndcultureCode/AndcultureCode/issues/108

### Learning Boundaries & Tests

It is common to write some test code when exploring a new library - it's a quick, easy way to familiarize
yourself with the API. They can also be a valuable tool for ensuring expected behavior of the library
is maintained between releases.

While you shouldn't unit test 100% of the library's API, there are certain scenarios where you'd
like to be notified with a failing test if an upgrade changes the assumptions your code is written with.

For example, `immutable`, the library we use for constructing view models on the frontend, currently
behaves in a way that requires us to pass an object with every single property set in order to be
"settable" when instantiated. Without doing so, the instance will return `undefined` for properties
not initially given, which is not obvious or expected behavior.

```ts
// interfaces/user-topic.ts
interface UserTopic {
    description?: string;
    name: string;
    userId: number;
}

// models/user-topic-record.ts
const defaultValues: UserTopic = {
    // Forgot to set "description" here, which should be okay - it's optional
    name: "",
    userId: 0,
};

class UserTopicRecord extends Record(defaultValues) implements UserTopic {}

// component.tsx
const record = new UserTopicRecord({
    name: "Other",
    description: "Miscellaneous items",
    userId: 123,
});

console.log(record.description); // Outputs `undefined`
```

To verify this behavior, we can write a test (example from [`AndcultureCode.JavaScript.Core`](https://github.com/AndcultureCode/AndcultureCode.JavaScript.Core) -> `record-utils.test.ts`)

```ts
test("when property values not provided in class definition, returns record without properties set", () => {
    // Arrange

    // Because we're not sending through values (even undefined) to the class definition
    // they will not be set even when supplied during construction. If immutable ever
    // changes this behavior, this test should let us know
    class UserRecord extends Record({}) implements User {}

    // Act
    const sut = new UserRecord({
        createdById: TestUtils.faker.datatype.number(),
        createdOn: TestUtils.faker.datatype.string(),
        deletedById: TestUtils.faker.datatype.number(),
        deletedOn: TestUtils.faker.datatype.string(),
        updatedById: TestUtils.faker.datatype.number(),
        updatedOn: TestUtils.faker.datatype.string(),
    });

    // Assert
    expect(sut.createdById).toBeUndefined();
    expect(sut.createdOn).toBeUndefined();
    expect(sut.deletedById).toBeUndefined();
    expect(sut.deletedOn).toBeUndefined();
    expect(sut.updatedById).toBeUndefined();
    expect(sut.updatedOn).toBeUndefined();
});
```

### Using Code That Does Not Yet Exist

https://github.com/AndcultureCode/AndcultureCode/issues/110

### Clean Boundaries

Change is one of the things that happens at boundaries, and good software design can accommodate
change without large investments of time and work.
When we use code from other teams, we need to be careful to protect our investment and make sure
that future changes are not too costly.
Code at boundaries needs to be clearly separated and our tests need to define our expectations.
Our code should not know the particulars of external code, and boundaries allow us to interact with
code and standards that we do control and understand.
