# Constraints and limitations

The test platform has been designed to be versatile and developer-friendly. Some constraints and limitations remain, the most important being:

- The virtual device has some limitations like networking, we're trying to add as many features as possible.

- Hardware-specific code (to be tested on real hardware) and generic code (to be tested on a virtual device) cannot be mixed in a class library because of the specialized (native) assemblies required for the hardware specific code.

- If unit tests assemblies and test framework extensions are dependent on the same class library, they should use the same (or at least a compatible) version of the class library. The test platform cannot handle different versions of the same library.

- There is no way to validate that for a serial port the deployment configuration and the connected device match. This is the responsibility of the tester/developer.

- The output directory where the test assembly ends up has to be in one of the sub folders of the `nfproj` project directory. By default it is in `nfproj_project_directory/bin/Debug`. So do not adjust the default settings, they'll work perfectly for our use cases.

- The .runsettings configuration for Visual Studio/VSTest is created by the test platform and contains absolute paths. The location where NuGet packages are stored and the `ResultsDirectory` should have absolute paths of less than 255 characters in length.
