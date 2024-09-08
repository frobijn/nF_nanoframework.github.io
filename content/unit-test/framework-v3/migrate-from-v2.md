# Migrate from nanoFramework.TestFramework v2

## Keep using the v2 test platform
It is possible to have a solution with both unit tests project that use v2 of the test platform and projects that use v3. The v2 projects will be handled by the v2 test adapter, the v3 projects by the v3 test platform. The v2 test projects will not benefit from any of the v3 improvements.

## Migrate to v3, do not update any code
The next step is to migrate the v2 test projects to v3:

- Remove the NuGet package `nanoFramework.TestFramework` from the project and add `nanoFramework.TestFramework.UnitTestsProject`.

- Rebuild the unit tests project.

The first time the unit test project is (re)built, the v3 test platform updates the project:

- The fixed assembly name `NFUnitTest` is removed from the project; the test assembly will be named after the project.

- The project file is modified to use the v3 `nano.vstest.runsettings` file.

- The `nano.runsettings` file is migrated to a v3 `nano.runsettings`/`nano.runsettings.user` pair and a `nano.vstest.runsettings` file. If there are no settings to store in one of the files, that file is not created/removed. Be aware the the v2 `RealHardwarePort` setting is migrated to the `nano.runsettings.user` file that is no longer stored in the git repository.

The unit tests will be run using the v3 test platform. To make maximum use of running tests in parallel, consider moving the `nano.vstest.runsettings` to the solution directory or to the top of the repository, and add the corresponding `GlobalSettings` element to `nano.runsettings` as per [specification](controlling-the-test-execution#configuration-file-hierarchy).

It is now also possible to use [debug unit tests projects](debugging-unit-tests).

## Reorganize the .runsettings and use assembly attributes

It is recommended that the next step is to start using the [.runsettings hierarchy](controlling-the-test-execution#configuration-file-hierarchy) and move all `nano.runsettings`/`nano.runsettings.user`/`nano.vstest.runsettings` from the project directory to the solution directory or a global directory in the repository.

The project-specific `nano.runsettings` file limit the execution of the unit tests to a virtual device or to real hardware. Add `[TestOn...]` [assembly attributes](writing-unit-tests#assembly-and-test-class-attributes) to the code of the test project to achieve the same.

## Start using the new v3 features

Start adding the v3 attributes to the code of the unit test project when you are ready to do so.