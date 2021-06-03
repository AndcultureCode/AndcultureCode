# Clean Code: Error Handling

[Table of Contents](../CLEAN-CODE.md)

## Use Exceptions Rather than Error Codes

Generally, exceptions are easier to maintain in comparison to error codes, so we prefer to use them in our applications.
Code that uses error codes can be greatly simplified through the use of a `try-catch` block returning a typed exception and a descriptive error message.

```CSharp
public class DeviceController
{
    public void SendShutDown()
    {
        DeviceHandle handle = GetHandle(DEV1);
        //Check the state of the device
        if (handle != DeviceHandle.INVALID)
        {
            // Save device status to the record field
            GetDeviceRecord(handle);
            //If not suspended, shut down
            if (record.GetStatus() != DEVICE_SUSPENDED)
            {
                PauseDevice(handle);
                ClearDeviceWorkQueue(handle);
                CloserDevice(handle);
            }
            else
            {
                _logger.Log($"Device suspended. Unable to shut down.");
            }
        }
        else
        {
            _logger.Log($"Invalid Handle for:{DEV1.ToString()}");
        }
    }
}
```

In practice, we tend to follow stricter policy regarding when an `Exception` should be used. In the normal flow of a program, we prefer to bubble up errors through the `IResult<T>` pattern and only throw exceptions when an execution failure occurs.

```CSharp
public class DeviceController
{
    public const string ERROR_KEY_DEVICE_SHUTDOWN_FAILURE = "DeviceController.Shutdown.Failure";
    ...
    private ILogger<DeviceController> _logger;
    ...

    public void SendShutDown()
    {
        var shutDownResult = ShutDown();
        if (shutDownResult.HasErrors)
        {
            _logger.LogErrors(shutDownResult.Errors);
        }
    }

    private IResult<bool> ShutDown()
    {
        var deviceHandleResult = GetHandle(DEV1);
        if (deviceHandleResult.HasErrorsOrResultIsNull())
        {
            r.AddErrors(deviceHandleResult);
            return false;
        }

        var handle = deviceHandleResult.ResultObject;
        var deviceRecordResult = GetDeviceRecord(handle);
        if (deviceRecordResult.HasErrorsOrResultIsNull())
        {
            r.AddErrors(deviceRecordResult);
            return false;
        }

        var pauseResult = PauseDevice(handle);
        if (pauseResult.HasErrors)
        {
            // Add Errors and Continue
            r.AddErrors(pauseResult);
        }

        var clearResult = ClearDeviceWorkQueue(handle);
        if (clearResult.HasErrors)
        {
            // Add Errors and Continue
            r.AddErrors(clearResult);
        }

        var closeResult = CloseDevice(handle);
        if (closeResult.HasErrors)
        {
            // Add Errors and Continue
            r.AddErrors(closeResult);
        }

        return true;
    }

    private IResult<DeviceHandle> GetHandle(DeviceId id) => Do<DeviceHandle>().Try((r) =>
    {
        if (string.isNullOrWhitespace(id))
        {
            throw new ArgumentNullException("Device Handle was unexpectedly null or empty");
        }
        ...
        ...
        return deviceHandle;
    });
}
```

## Write Your Try-Catch-Finally Statement First

Failure scenarios should be thought about up front to build reliable software. Too often we are
tempted to simply write the happy-path, leaving our applications to terminate normal execution with
unhandled exceptions.

Following Microsoft's recommendations to avoid exceptions for normal code flow, we draw inspiration
from their .NET Framework `TryXXX()` Pattern by using [`Do.Try`](https://andculturecode.github.io/AndcultureCode.CSharp.Core/docs/#dotry).

```csharp
public IResult<List<Role>> FindExternalRoles(long userId) => Do<List<Role>>.Try((r) =>
{
    var externalRoles = _thirdPartyProvider.FindRoles(userId);
    // additional logic around returned roles
    return externalRoles.MapToRoles();
})
.Catch(HandleAggregateException)
.Result;

public IResult<List<User>> FindExternalUsers(string searchText) => Do<List<User>>.Try((r) =>
{
    var externalUsers = _thirdPartyProvider.FindUsers(searchText);
    // additional logic around returned users
    return externalUsers.MapToUsers();
})
.Catch(HandleAggregateException)
.Result;

private void HandleAggregateException(AggregateException ex, IResult<bool> result)
    => result.AddErrorAndLog(_logger, _localizer, ERROR_EXTERNAL_API, ex);
```

Leveraging C# expression bodies, a sprinkling of functional programming and the idea that [exceptions should
be exceptional](https://mattwarren.org/2016/12/20/Why-Exceptions-should-be-Exceptional/), we aim to
catch unhandled exceptions where they occur and gracefully translate those errors in a properly
encapsulated fashion before letting them flow through or around the system to layers that would otherwise
have zero context.

## Define Exception Classes in Terms of a Caller's Needs

Errors can be classified in multiple ways, such as by source (what component caused it) or
by type (device failure vs. network failure vs. programming errors).
However, we should define exception classes by how they are caught.
It is not beneficial to throw 5 different kinds of exceptions when the caller only cares about
2 types of exceptions.

When working with third-party APIs it is a best practice to wrap their API in your own class.
This allows you to convert their error handling into exceptions that make sense for your application,
rather than passing along third-party exceptions.
This minimizes dependencies and allows you to more easily handle API changes.
It also makes mocking third-party calls easier when testing code.
