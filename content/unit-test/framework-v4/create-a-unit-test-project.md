# Create a unit test project

## Create a testable project

Before even considering to create a unit test project, you should consider whether the project(s) to create the unit tests for satisfies the criteria of a testable project:

- The project is a class library

	It is not possible to create unit tests for .NET nanoFramework applications. Instead, create a class library first, move all testable code to the library and create a unit test project for that library.

- The project is suited for the device you want to test on

	It is possible to run and debug unit tests on both the virtual device and on real hardware. The virtual device runs on your computer or in an automated build/test environment without additional hardware. Running tests on the virtual device is very speedy as the simultaneous execution of multiple test projects can be optimized. If you have code that does not directly require additional hardware, e.g., for additional processing of the data obtained from the hardware, it is advantageous to test and debug the code on the virtual machine. Even if the code can also be tested as part of a regular .NET assembly, as there are subtle differences in the implementation of core .NET features in .NET **nanoFramework** and regular .NET platforms.

	Using the virtual device has a major limitation: it is not possible to debug a unit test on the virtual machine if any code of the unit test project, the class library under test or any of its dependencies has hardware-specific features. Not even if none of the hardware-specific code is (indirectly) invoked in the code or unit test you want to run on the virtual device. It simply is not possible to deploy the assemblies to the virtual device for debugging. You may find it possible to run such unit tests on the virtual device, but if the unit test fails and you want to start the debugger to investigate why it fails, you're out of luck.

	It is recommended to separate code that can be tested/debugged on the virtual device and hardware-specific code into two separate class libraries. Make sure the hardware-specific code library has the generic-code-library as a dependency and not the other way round. E.g., if you define interfaces as an abstraction for the hardware-specific code, make sure the interfaces are in the generic code library. Then create two unit test projects, one for each class library. Test and debug the generic code library primarily on the virtual machine (but make sure it runs on real hardware as well) and the hardware-specific library on real hardware.

	There is no technical objection to make the class libraries and unit test projects part of the same solution. In fact that may be the best way to develop both class libraries as one entity.

## Create a new unit test project

The easiest way to create a new unit test project is to use the project templates provided by the Visual Studio extension. 

- Select *File | New | Project* in Visual Studio

- Filter the project templates by platform: *nanoFramework*

- Select *Unit Test Project (.NET nanoFramework)*

After the project is created, add the .NET nanoFramework class library that you want to test as a reference.

## Create as class library project

A unit test project is just a .NET nanoFramework class library that references the `nanoFramework.TestFramework` NuGet package. If that suites you better, you can start by creating a class library project and then add the NuGet package.
