# Run tests in VSTest

The tests in a test assembly can also be run using command line tools or as part of a CI/CD pipeline. The test platform is controlled from one of the VSTest-family of applications, e.g.:

- [vstest.console.exe](https://learn.microsoft.com/en-us/visualstudio/test/vstest-console-options?view=vs-2022)
- [VSTest@2](https://learn.microsoft.com/en-us/azure/devops/pipelines/tasks/reference/vstest-v3?view=azure-pipelines) or [VSTest@3](https://learn.microsoft.com/en-us/azure/devops/pipelines/tasks/reference/vstest-v4?view=azure-pipelines) task in Azure DevOps

The details are slightly different for each application. But there are a few things that you have to do to enable the use of the test platform: 

- Make sure the .NET **nanoFramework** tests can be selected separately from tests using another test framework.

- Pass the configuration of the .NET **nanoFramework** test adapter.

- Select the tests for which nanoDevices are available.

## Make nanoFramework tests recognizable.

The first item is relevant if it is possible to specify test assemblies using a pattern (e.g., `**\*test*.dll`). The VSTest@2/VSTest@3 Azure DevOps tasks support that, but `vstest.console.exe` does not. A good way to make the tests recognizable is by a project name convention. E.g., project names that end in `NFTests` are always .NET **nanoFramework** test assemblies; these can be selected with a pattern like `**\*.nftests.dll` and that pattern can be excluded when running tests using other frameworks.

The reason this is necessary has to do with the other things you have to do: most likely you are going to provide a **nanoFramework**-specific configuration and are selecting the tests to run. VSTest works great if its configuration is shared by all tests. If the configuration does not specify which test adapter to use and the test adapters can be auto-discovered, a single VSTest instance can handle multiple .NET architectures and test frameworks. If anything has to be configured, the configuration applies to all tests executed by the VSTest instance and multi-test-framework support is much harder to realize. A single VSTest instance can have one filter to select tests, but the syntax is different for the various test frameworks and VSTest cannot handle differences in syntax. So while it may be possible in some cases to configure the test adapter and select tests in a way that is compatible with tests created for other frameworks, it is not easy and it is likely that there will be a time it is not possible at all.

## Configuration of the test adapter

VSTest has to know where the .NET **nanoFramework** test adapter is located and how to run it. The easiest way is to pass a *.runsettings* [configuration file](controlling-the-test-execution.md#configuration-of-the-vstest-test-host). For each test project such a file is created in the output directory, e.g.:

```
vstest.console.exe /Settings:TestProject\bin\Release\nano.vstest.runsettings TestProject\bin\Release\TestProject.dll
```
or use that file in the task, e.g.:

```
  - task: VSTest@2
    condition: succeeded()
    displayName: 'Running tests'
    inputs:
      testSelector: 'testAssemblies'
      testAssemblyVer2: |
        **\*.NFTests*.dll
        !**\obj\**
      runSettingsFile: 'TestProject\bin\Release\nano.vstest.runsettings'
      ...
``` 

Do not use the `nano.vs.runsettings` to run the tests via VSTest in a CI/CD pipeline if tests have to be run on hardware nanoDevices. The difference between the two files is the `Scheduling` setting that determines how tests are scheduled. It is set to `Interactive` in `nano.vs.runsettings` and to `Sequential` in `nano.vstest.runsettings`. The latter is first-come-first-served scheduling: if multiple tests need to be run on the same hardware nanoDevice, the test that is scheduled first is run first. Tests scheduled for an *Interactive* setting take precedence over *Sequential* test, and the order in which tests are run depends on the number of devices a test can be run on. The *Interactive* scheduling tries to optimize the total test time, e.g., by prioritizing test projects that can only run on a particular nanoDevice, even if they are scheduled at a later time. If *Interactive* scheduling is used in a CI/CD pipeline where new tests are scheduled all the time, it may occur that some tests are never run as newly scheduled tests keep being prioritized.

## Select the tests for the available nanoDevices

A CI/CD pipeline most likely runs in an environment where no real hardware nanoDevices are available. And most likely some of your tests will require such a device. If VSTest is run without test case filter, these tests will be reported as skipped and may give a false impression that there's something wrong with your code. It is better to instruct VSTest not to run the tests at all.

VSTest supports the election of tests that should (not) be run via a test case filter (`vstest.console.exe /TestCaseFilter` or `VSTest@2 testFiltercriteria`). The [syntax](https://github.com/microsoft/vstest/blob/main/docs/filter.md) for the filter is the same for all test frameworks, except for the supported properties. The .NET **nanoFramework** test adapter supports the properties:

- `FullyQualifiedName` is the fully qualified name of a test case: *namespace.testclassname.testmethodname*.
- `ClassName` is the fully qualified name of a test class: *namespace.testclassname*.
- `MethodName` is the name of a test method
- `DisplayName` is the name of the test case as shown in the Visual Studio Test Explorer that has the general form: *name(data_row_arguments) [nanoDevice name]*.
- `TestCategory` or `Trait` is a category assigned to the test method via the `[TestCategory]` attributes (or any attribute implementing the `ITestCategory` interface), including the categories that start with `@` (e.g., `@Virtual nanoDevice`) that are automatically added by the test platform.

In a CI/CD pipeline without hardware nanoDevices you would specify the filter `TestCategory=@Virtual nanoDevice` or `TestCategory!=@Hardware nanoDevice` to select only the tests that should be executed on a Virtual nanoDevice. E.g.:

```
  - task: VSTest@2
    condition: succeeded()
    displayName: 'Running tests'
    inputs:
      testSelector: 'testAssemblies'
      testAssemblyVer2: |
        **\*.NFTests*.dll
        !**\obj\**
      testFiltercriteria: 'TestCategory!=@Hardware nanoDevice'
      ...
```
You can create a script to run all relevant tests on real hardware on a developer/tester's computer by selecting the tests:
```
vstest.console.exe "/TestCaseFilter:TestCategory=@Hardware nanoDevice" /Settings:... ...test-dll...
```
The *@Hardware nanoDevice* category is automatically added to all tests that should be run on real hardware, even if the test is specified to run on a particular platform or target.
