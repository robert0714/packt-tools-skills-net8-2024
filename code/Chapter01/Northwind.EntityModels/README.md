# Creating a class library for entity models using SQL Server
You will now create the entity models using the dotnet-ef tool:

1. Using your preferred code editor, create a new project, as defined in the following list:
   * Project template: **Class Library** / `classlib`
   * Project file and folder: `Northwind.EntityModels`
     ```bash
     dotnet new classlib -n Northwind.EntityModels
     ```
2. In the `Northwind.EntityModels` project, treat warnings as errors, and add package references for the SQL Server database provider and EF Core design-time support, as shown highlighted in the following markup:
   ```xml
   <Project Sdk="Microsoft.NET.Sdk">
        <PropertyGroup>
            // ...existing code...
            <TreatWarningsAsErrors>true</TreatWarningsAsErrors>
            // ...existing code...
        </PropertyGroup>
   // ...existing code...
   </Project>
   ```
   cli:
   ```bash
   dotnet add package Microsoft.EntityFrameworkCore.SqlServer --version 8.0.20
   dotnet add package Microsoft.EntityFrameworkCore.Design --version 8.0.20
   ```
3. Delete the `Class1.cs` file.
4. Build the `Northwind.EntityModels` project.
5. Open a command prompt or terminal for the `Northwind.EntityModels` folder.
6. If you do not already have the `dotnet-ef` tool, then install the latest version, as shown in the following command:
   ```bash
   dotnet tool install --global dotnet-ef  --version 8.0.20 
   ```
7. At the command line, generate entity class models for all tables, as shown in the following commands:
   * cmd
     ```bash
     dotnet ef dbcontext scaffold "Server=127.0.0.1,1433;Database=Northwind;User Id=sa;Password=StrongP@ssw0rd123!;TrustServerCertificate=True;MultipleActiveResultSets=True;" Microsoft.EntityFrameworkCore.SqlServer --namespace Northwind.EntityModels --data-annotations
     ```
   * PowerShell
     ```bash
     dotnet ef dbcontext scaffold "Server=127.0.0.1,1433;Database=Northwind;User Id=sa;Password=StrongP@ssw0rd123!;TrustServerCertificate=True;MultipleActiveResultSets=True;" Microsoft.EntityFrameworkCore.SqlServer `
     --namespace Northwind.EntityModels `
     --data-annotations `
     --force
     ```

8. Note that 28 classes were generated, from `AlphabeticalListOfProduct.cs` to `Territory.cs`.
9. At the top of the [`NorthwindContext.cs`](../Northwind.DataContext/NorthwindContext.cs) file, import the namespace for working with ADO.NET types, as shown in the following code:
    ```csharp
    using Microsoft.Data.SqlClient; // To use SqlConnectionStringBuilder.
    ```
10. Modify the `OnConfiguring` method to dynamically set the connection string and set any sensitive parameters using environment variables, as shown in the following code:
    ```csharp
    protected override void OnConfiguring(
    DbContextOptionsBuilder optionsBuilder)
    {
        // If not already configured by a client project. For example,
        // a client project could use AddNorthwindContext to override
        // the database connection string.

        if (!optionsBuilder.IsConfigured)
        {
        SqlConnectionStringBuilder builder = new();

        builder.DataSource = ".";
        builder.InitialCatalog = "Northwind";
        builder.TrustServerCertificate = true;
        builder.MultipleActiveResultSets = true;

        // If using Azure SQL Edge.
        // builder.DataSource = "tcp:127.0.0.1,1433";

        // Because we want to fail fast. Default is 15 seconds.
        builder.ConnectTimeout = 3;

        // If using Windows Integrated authentication.
        builder.IntegratedSecurity = true;

        // If using SQL Server authentication.
        // builder.UserID = Environment.GetEnvironmentVariable("MY_SQL_USR");
        // builder.Password = Environment.GetEnvironmentVariable("MY_SQL_PWD");

        optionsBuilder.UseSqlServer(builder.ConnectionString);
        }
    }
    ```
11. In `Customer.cs`, the dotnet-ef tool correctly identified that the `CustomerId` column is the primary key and it is limited to a maximum of five characters, but we also want the values to always be uppercase. So, add a regular expression to validate its primary key value to only allow uppercase Western characters, as shown highlighted in the following code:
    ```csharp
    [Key]
    [StringLength(5)]
    [RegularExpression("[A-Z]{5}")] 
    public string CustomerId { get; set; } = null!;
    ```