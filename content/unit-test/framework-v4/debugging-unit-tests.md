# Debug unit tests


## How to debug unit tests

If a unit test fails unexpectedly and it is not obvious why, the best approach is to start the debugger and look what is happening in the test. The .NET **nanoFramework** test platform supports this, both on the virtual machine and on real hardware. But it is not possible to start the debugger directly from the Test Explorer in Visual Studio.

Instead the test platform makes it easy to debug one or more unit tests from a separate project. The steps involved are:

- Create a debug-project per unit test project, or for multiple unit test projects as long as they will be debugged on the same device (cf. [testable projects](create-a-unit-test-project#create-a-testable-project)). You have to do this only once per unit test project.

- Select the unit test(s) to debug

- Set breakpoints in the unit tests

- Start the debugger and deploy the project to the correct device.

The test platform will generate the code for setup, cleanup and for calling the test methods exactly as they are executed in the unit test project.

## Create the debug project

Create a new unit test project using the project templates provided by the Visual Studio extension. 

- Select *File | New | Project* in Visual Studio

- Filter the project templates by platform: *nanoFramework*

- Select *Unit Test Debug Project (.NET nanoFramework)*

This will create a new application project.

- Add the unit test project(s) as a reference to the new project.

- Build the project; this will generate some additional files.

The project now contains a json file that is used to specify which unit tests to debug: `SelectUnitTests.json`. In this file you as a developer specify which unit test(s) you want to debug. This file is excluded by default from the git repository using a local `.gitignore` file, and are re-generated if they are not present. The rationale is that the selection of unit tests is an ad-hoc decision that varies a lot. You are of course free to remove the `.gitignore` file.  

## Select the unit test(s) to debug

Open the `SelectUnitTests.json` file in Visual Studio. The editor offers intellisense when typing the specification. An example of the specification is:

```json
{
  "$schema": "obj/nF/SelectUnitTests.schema.json"

  "TestCases": {

    "nanoFramework.TestFramework.Test": {

      "TestOfTests": [

        "TestMethod",
        { "TestMethodWithDataRows": [ 0, 1 ] },
        { "OtherTestMethodWithDataRows": "*" }

      ],

      "OtherTestClass": "*"

    }
  }
}
```

- The test cases are selected by first specifying the namespace, then the test class, then the test methods. To select all test methods of a test class, specify `*` instead of an array of test methods.

- If a test method has `[DataRow]` attributes, each attribute corresponds to a separate test case. Specify the name of the test method and an array of the 0-based index of the attributes to include. If all data row attributes should be included, just specify `*` instead of the array.

- If the intellisense (that uses the JSON schema) does not know a namespace, test class, test method or data attribute, rebuild the project and the names should be known.

- The file can also contain information about the [deployment configuration](deployment-configuration).

When the project is built, the test platform tooling will generate the code to run the unit tests. The `[TestOn...]` [attributes](writing-unit-tests#where-to-run-a-test-method-device-selection) are ignored. You have to make sure that the correct device is selected when deploying the project for debugging.

