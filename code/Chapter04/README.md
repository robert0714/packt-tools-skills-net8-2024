## Debugging test projects
1. Use your preferred code editor to add a new **xUnit Test Project** / `xunit` project named `DebugTests` to the `Chapter04` solution.
    ```bash
    dotnet add package xunit
    dotnet add package xunit.runner.visualstudio
    ```
2. In the `DebugTests.csproj` project file, add a project reference to the class library, as shown in the following markup:
    ```xml
    <ItemGroup>
    <ProjectReference Include="..\DebugLibrary\DebugLibrary.csproj" />
    </ItemGroup>
    ```
3. Build the `DebugTests` project.
4. In the `DebugTests` project, rename `UnitTest1.cs` to `DebugLibraryTests.cs`.
5. In `DebugLibraryTests.cs`, write statements in a test method to instantiate a `WebConfig` object and check that its properties are not `null`, as shown in the following code:
    ```csharp
    using Packt.Shared; // To use WebConfig.
    namespace `DebugTests`;
    public class DebugLibraryTests
    {
        [Fact]
        public void WebConfigPropertiesInstantiated()
        {
            WebConfig config = new();
            Assert.NotNull(config.DbConnectionString);
            Assert.NotNull(config.Base64Encoded);
            Assert.NotNull(config.JsonWebToken);
        }
    }
    ```
6. Set a breakpoint on the first assertion.
7. Navigate to Test | Debug All Tests.