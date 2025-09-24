# Creating a test project to check the integration of the class libraries
Since we will not be creating a client project in this chapter that uses the EF Core model, we should create a test project to make sure the database context and entity models [integrate correctly](https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-csharp-with-xunit):

1. Use your preferred code editor to add a new **xUnit Test Project [C#]** / `xunit` project named `Northwind.Tests` to the `Chapter01` solution.
   ```bash
   dotnet new xunit -o Northwind.Tests
   ```  
2. In `Northwind.Tests.csproj`, modify the configuration to treat warnings as errors and add an item group with a project reference to the `Northwind.DataContext` project, as shown in the following markup:
    ```xml
    <ItemGroup>
      <ProjectReference Include=
        "..\Northwind.DataContext\Northwind.DataContext.csproj" />
    </ItemGroup>
    ```
   Its cli:
    ```bash
    dotnet add ./Northwind.Tests/Northwind.Tests.csproj reference ./Northwind.DataContext/Northwind.DataContext.csproj
    ```
3. Build the `Northwind.Tests` project to build and restore project dependencies.
4. Rename the file `UnitTest1.cs` to `NorthwindEntityModelsTests.cs` (Visual Studio prompts you to rename the class when you rename the file).
5. In `NorthwindEntityModelsTests.cs`, if you are using Code, then manually rename the class to `NorthwindEntityModelsTests`.
6. In `NorthwindEntityModelsTests.cs`, modify the class to import the Northwind.EntityModels namespace and have some test methods for ensuring that the context class can connect, the provider is SQL Server, and the first product is named `Chai`, as shown in the following code:
    ```csharp
    using Northwind.EntityModels; // To use NorthwindContext and Product.

    namespace Northwind.Tests;

    public class NorthwindEntityModelsTests
    {
      [Fact]
      public void CanConnectIsTrue()
      {
        using (NorthwindContext db = new()) // arrange
        {
          bool canConnect = db.Database.CanConnect(); // act
          Assert.True(canConnect); // assert
        }
      }

      [Fact]
      public void ProviderIsSqlServer()
      {
        using (NorthwindContext db = new())
        {
          string? provider = db.Database.ProviderName;
          Assert.Equal("Microsoft.EntityFrameworkCore.SqlServer", provider);
        }
      }

      [Fact]
      public void ProductId1IsChai()
      {
        using (NorthwindContext db = new())
        {
          Product? product1 = db.Products?.Single(p => p.ProductId == 1);
          Assert.Equal("Chai", product1?.ProductName);
        }
      }
    }

    ```
## Running tests
* Using Code:
  1. In Code, in the `Northwind.Tests` project’s **TERMINAL** window, run the tests, as shown in the following command:
     ```bash
     dotnet test 
     ```
  2. In the output, note that the results indicate that three tests ran, and all passed.
  >  If you are using **C# Dev Kit**, then you can also build the test project and then run the tests from the **Testing** section in the **Primary Side Bar**.
        