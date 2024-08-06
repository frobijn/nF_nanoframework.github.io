# Use custom tooling

## Why custom tooling may be useful

The .NET **nanoFramework** test platform is designed to create automated tests that can run on a virtual device or real hardware without human intervention. Not all software or (custom) hardware can be tested via automated tests. For example: if you are working on an appliance that has a small graphical touchscreen for user interaction, it is not possible to use the test platform to create an automated test to verify whether the screen shows the correct information, or whether the software correctly identifies a symbol touched by the user. It still is very convenient to create (unit) tests using the test platform: if the software and/or hardware has been changed in development or product maintenance, a tester can systematically test the modified component and use failed unit tests to investigate and fix bugs. Either you have to provide some manual assistance, or maybe you have custom test-hardware to probe the screen. And you'll have some code that runs on the nanodevice to let a test interact with the tester (e.g., via a website running on the nanodevice) or test-hardware.

Such tests typically require additional resources, e.g., the files required for the website. In regular .NET test projects, the resources could be added to the test assembly and discovered at runtime. It is perfectly possible to create a `TesterGuidance` component as extension to the test platform that can be used in tests, e.g.:
```csharp
[TestClass]
public class TouchscreenTests
{
    [TestMethod]
    public void IdentifyTouchedSymbolTest ()
    {
        var guidance = new TesterGuidance();

        // .. show symbols ..

        while (guidance.TesterWantsToContinue ("webpage_name"))
        {
            // .. identify symbol that is touched

            guidance.TesterShouldAssert (/* .. symbol found .. */);
        }
    }
}
```
The implementation of `TesterGuidance` can use call stack analysis to find the test method it is called from and use reflection to find all html/css/script files associated with the *webpage_name_* included as embedded resource. But many of the required .NET analysis features are not implemented (by design) in the .NET **nanoFramework**, as they are not used in the production version of the software.

It is possible to create a similar extension for the .NET **nanoFramework** test platform, but all of the analysis/collection of resources/... has to be done by a custom tool that generates code to be included in the test project. As an example, the `TesterGuidance` component could be designed to use a mix of attributes and test platform extensions:
```csharp
[TestClass]
public class TouchscreenTests
{
    [TestMethod]
    [TesterGuidance(typeof (TouchscreenTests), nameof (IdentifyTouchedSymbolTest), "webpage_name")]
    public void IdentifyTouchedSymbolTest (ITesterGuidance guidance)
    {
        // .. show symbols ..

        while (guidance.TesterWantsToContinue())
        {
            // .. identify symbol that is touched

            guidance.TesterShouldAssert(/* .. symbol found .. */);
        }
    }
}
```
The `TesterGuidanceAttribute` has two functions. It acts as a marker for a custom tool to generate a class and maybe even a resource file to add all required files to the project:
```csharp
internal class TouchscreenTests_IdentifyTouchedSymbolTest_TesterGuidance : ITesterGuidance
{
    private string GetResource(string relativeUrl)
    {
        if (relativeUrl == "index.html")
        {
            return TesterGuidance1Resources.GetString(TesterGuidance1Resources.StringResources.webpage_name_html);
        }
        else if (relativeUrl == "index.css")
        {
            return TesterGuidance1Resources.GetString(TesterGuidance1Resources.StringResources.webpage_name_css);
        }
        // ...
    }

    // ... more code...
}
```
The attribute implements `IDataRow` to pass the instance of the guidance to the test method:
```csharp
public class TesterGuidanceAttribute: Attribute, IDataRow
{
    public TesterGuidanceAttribute (Type testClassType, string testMethodName, string webPageName)
    {
        // Omitted for brevity
    }

    public Type TestClassType { get; }
    public string TestMethodName { get; }
    public string WebPageName { get; }

    object[] IDataRow.MethodParameters
    {
        get
        {
            Type guidanceType = TestClassType.Assembly.GetType($"{TestClassType.FullName.Replace('.','_')}.{TestMethodName}_TesterGuidance");
            object guidance = Type.GetConstructor(new Type[0]).Invoke (new object[0]);
            return new object[] { guidance };
        }
    }
}
```

## Custom tool that analyses a test assembly

There is some similarity between the custom tool to generate code in the example above, and the tools in the .NET **nanoFramework** test platform. Both have to analyse the test cases that are present in the test assembly, and act on the attributes for a test method, test class and test assembly. The components that do the heavy lifting for the test platform are available in the `nanoFramework.TestFramework.Tooling` package. A possible implementation of the custom tool is to use this package: analyse a test assembly after it has been build, and create or update the test guidance class and resources if necessary.

To use the `nanoFramework.TestFramework.Tooling` package, the custom tool has to be a .NET Framework 4.8 application. Given the path to the test assembly, it can get a list of test cases:

```csharp
var testCases = new TestCaseCollection("assembly_path.dll", true, (a) => ProjectSourceInventory.FindProjectFilePath(a, logger), logger);
```
A test case corresponds to a test method (if there are attribute that supports `IDataRow`: one test case per attribute), a test class to a test case group. Custom attributes for test methods are accessible via `TestCase.CustomAttributeProxies`, for test class via `TestCaseGroup.CustomAttributeProxies`, and for an assembly via `TestCaseSelection.CustomAttributeProxies`.

There's one catch: the custom attribute types (like `[TesterGuidance]`) are not recognized in the custom tool, as they are defined in an assembly that is based on .NET **nanoFramework** instead of .NET Framework. For the .NET infrastructure the `typeof (TesterGuidanceAttribute)` in the custom tool is different from `[TesterGuidance]` in the test assembly. Instead the custom tool should declare a proxy:
```
public class TesterGuidanceAttributeProxy : AttributeProxy
{
    private readonly object _attribute;
    private readonly TestFrameworkImplementation _framework;

    public TesterGuidanceAttributeProxy(object attribute, TestFrameworkImplementation framework, Type attributeType)
    {
        _attribute = attribute;
        _framework = framework;
        framework.AddProperty<Type>('<namespace>.TesterGuidanceAttribute', attributeType, "TestClassType");
        framework.AddProperty<string>('<namespace>.TesterGuidanceAttribute', attributeType, "TestMethodName");
        framework.AddProperty<string>('<namespace>.TesterGuidanceAttribute', attributeType, "WebPageName");
    }

    public Type TestClassType
        => _framework.GetPropertyValue<Type>('<namespace>.TesterGuidanceAttribute', "TestClassType", _attribute);

    public string TestMethodName
        => _framework.GetPropertyValue<string>('<namespace>.TesterGuidanceAttribute', "TestMethodName", _attribute);

    public string WebPageName
        => _framework.GetPropertyValue<string>('<namespace>.TesterGuidanceAttribute', "WebPageName", _attribute);
}
```
and register the proxy before investigating the test assembly:
```
AttributeProxy.Register('<namespace>.TesterGuidanceAttribute', false, (a,f,t) => new TesterGuidanceAttributeProxy(a,f,t));
```
Instances of the `TesterGuidanceAttributeProxy` will be present in the `CustomAttributeProxies` collections.