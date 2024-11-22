# Asserting in the test functions

As for most of the famous .NET Unit Test platform, the concept of `Assert` is present as well in .NET **nanoFramework**. You can see in the previous example some of those `Assert` functions. They take one or two arguments and are straight forward to use.

If there is an issue in those Assert function, an exception is raised. The test *pass* if there is **no** exception happening in the function. If any exception happens in the function, it is considered as *failed*.

Note that all the `Assert` functions can pass a custom message. For example: `Assert.Equal(42, 43, "My custom message saying that 42 is not equal to 43");`

The use of the `Assert` methods is illustrated in the example:

```csharp
namespace nanoFramework.TestFramework.Test
{
    [TestClass]
    public class TestOfTest
    {
        [TestMethod]
        public void TestStringComparison()
        {
            OutputHelper.WriteLine("Test string, Contains, EndsWith, StartWith");
            // Arrange
            string tocontains = "this text contains and end with contains";
            string startcontains = "contains start this text";
            string contains = "contains";
            string doesnotcontains = "this is totally something else";
            string empty = string.Empty;
            string stringnull = null;
            // Assert
            Assert.Contains(contains, tocontains);
            Assert.DoesNotContains(contains, doesnotcontains, "Your own error message");
            Assert.DoesNotContains(contains, empty);
            Assert.DoesNotContains(contains, stringnull);
            Assert.StartsWith(contains, startcontains);
            Assert.EndsWith(contains, tocontains);
        }

        [TestMethod]
        public void TestRaisesException()
        {
            OutputHelper.WriteLine("Test will raise exception");
            Assert.Throws(typeof(Exception), ThrowMe);
        }

        private void ThrowMe()
        {
            throw new Exception("Test failed and it's a shame");
        }
    }
}
```

## Assert.True and Assert.False

Simply check if something is True or False

```csharp
bool boola = true;
Assert.True(boola);
```

## Assert.Equal and Asset.NotEqual

`Assert.Equal` is  collection of functions that takes all the native Value Types as well as Array and check if the elements in the array are equals (if value) or same object (for non value types).

```csharp
Assert.Equal(elementa, elementb);
```

Same behavior for `Assert.NotEqual` but checking that the 2 elements are not equal.

```csharp
Assert.NotEqual(elementa, elementb);
```

## Assert.Null and Assert.NotNull

Those functions check that an element is null or not null.

```csharp
object objnull = null;
object objnotnull = new object();
Assert.Null(objnull);
Assert.NotNull(objnotnull);
```

## Assert.IsType and Assert.IsNotType

Those functions allows to check that an element is a specific type or not a specific type.

```csharp
Type typea = typeof(int);
Type typeb = typeof(int);
Type typec = typeof(long);
Assert.IsType(typea, typeb);
Assert.IsNotType(typea, typec);
```

## Assert.Empty and Assert.NotEmpty

## Assert.Same and Assert.NotSame

Functions to check that objects are the same or different.

```csharp
object obja = new object();
object objb = new object();
Assert.NotSame(obja, objb);
objb = obja;
Assert.Same(obja, objb);
```

## Assert for String checking

A set of functions to help checking strings is available. They allow most of the common scenarios, checking that a string contains specific elements, start with, end with as well as not containing some elements.

```csharp
// Arrange
string tocontains = "this text contains and end with contains";
string startcontains = "contains start this text";
string contains = "contains";
string doesnotcontains = "this is totally something else";
string empty = string.Empty;
string stringnull = null;
// Assert
Assert.Contains(contains, tocontains);
Assert.DoesNotContains(contains, doesnotcontains);
Assert.DoesNotContains(contains, empty);
Assert.DoesNotContains(contains, stringnull);
Assert.StartsWith(contains, startcontains);
Assert.EndsWith(contains, tocontains);
```

## Assert.Throws

This checks if a specific function will throw an exception. Usage:

```csharp
Assert.Throws(typeof(ExceptionTypeToCatch), AFunctionToCall);
```

Where:

- `ExceptionTypeToCatch` has to be a type of Exception. Typical example is to check if the function you're trying to call rases a `ArgumentException` for example.
- `AFunctionToCall` is an `Action`, so a function you can call to check if an exception is raised.

See the pattern in the previous example.

If an `Assert` fails in the `AFunctionToCall` or another of the test platform exceptions is thrown, that exception is passed on by `Assert.Throws` and the `Assert.Throws` itself does not fail or succeed. If `ExceptionTypeToCatch` is one of the test platform exceptions, the exception is not passed on and `Assert.Throws` will assert the result.

## Outputting messages from the tests

It's possible to output messages from the Unit Tests using `OutputHelper.Write` and `OutputHelper.WriteLine`. These work exactly as `Console.Write` and `Console.WriteLine` so simple or formatted output is available. `Debug.Write` and `Debug.WriteLine` are also supported.

```csharp
OutputHelper.WriteLine("This is a message from Unit Test XYZ!");
```

```csharp
OutputHelper.WriteLine($"This is another message from Unit Test XYZ, showing that {someVariable.Length} can be output too.");
```

## Abort a test for other reasons

If an expected condition is not met within a test method, that does not always mean the test should be marked as failed. There are other possible outcomes that are reported differently by the test platform. Example:

```csharp
namespace nanoFramework.TestFramework.Test
{
    [TestClass]
    public class TestOfTest
    {
        [TestMethod]
        public void MethodWillSkippIfFloatingPointSupportNotOK()
        {
            var sysInfoFloat = SystemInfo.FloatingPointSupport;
            if ((sysInfoFloat != FloatingPoint.DoublePrecisionHardware) && (sysInfoFloat != FloatingPoint.DoublePrecisionSoftware))
            {
                Assert.SkipTest("Double floating point not supported, skipping the Assert.Double test");
            }

            double on42 = 42.1;
            double maxDouble = double.MaxValue;
            Assert.Equal(42.1, on42);
            Assert.Equal(double.MaxValue, maxDouble);
        }

        #region Separate setup/cleanup methods
        [Setup]
        public void SetupSensorInSeparateMethod()
        {
            _sensor = MySensor.Initialize();   
            Assert.IsNotNull(_sensor, "Sensor cannot be initialized");
        }
        private MySensor _sensor;

        [TestMethod]
        public void TestSensorUsingSeparateSetupCleanupMethods()
        {
            Assert.IsTrue(_sensor.IsActive);
        }

        [Cleanup]
        public void CleanupSensorInSeparateMethod()
        {
            _sensor.Close ();
            Assert.IsFalse(_sensor.IsActive, "Sensor is still active");
        }
        #endif

        #region Setup/cleanup in test method
        [TestMethod]
        public void TestSensorWithSetupAndCleanupInMethod()
        {
            // Setup test context
            MySensor sensor = MySensor.Initialize();
            if (sensor is null)
            {
                Assert.SetupFailed("Sensor cannot be initialized");
            }
            Assert.IsNotNull(sensor, "Sensor cannot be initialized");

            // Perform the unit test
            Assert.IsTrue(_sensor.IsActive);

            // Cleanup
            sensor.Close ();
            if (!sensor.IsActive)
            {
                Assert.CleanupFailed("Sensor is still active");
            }
        }
        #endregion
    }
}
```

If a test cannot be run for some reason, use `Assert.SkipTest` in a test method. The test will be reported as *not run* instead of failed. If `Assert.SkipTest` is called in the constructor of the test class or in a setup method, all tests are skipped for which the constructor/setup method was called.

If a test method sets up a test context in the method (rather than via a constructor or setup method) and that fails, use `Assert.SetupFailed` to report that. The test will be shown as *failed*, but from the error message it will be clear that the setup has failed and not the test proper. It is not necessary to call `Assert.SetupFailed` in a constructor or setup method; any exception will be reported as a failed setup.


If a test method performs some cleanup in the method (rather than via `IDisposable` or a cleanup method) and that fails, use `Assert.CleanupFailed` to report that. The test will be shown as *failed*, but from the error message it will be clear that the cleanup has failed and not the test proper. It is not necessary to call `Assert.CleanupFailed` in `IDisposable.Dispose` or cleanup method; any exception will be reported as a failed cleanup.