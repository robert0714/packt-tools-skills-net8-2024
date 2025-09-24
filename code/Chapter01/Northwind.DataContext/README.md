# Creating a class library for the data context using SQL Server
Next, you will move the context model that represents the database to a separate class library:

1. Using your preferred code editor, create a new project, as defined in the following list:
   * Project template: **Class Library** / `classlib`
   * Project file and folder: `Northwind.DataContext`
     ```bash
     dotnet new classlib -n Northwind.DataContext
     ```
2. In the `Northwind.DataContext` project, add a project reference to the `EntityModels` project, and add a package reference to the EF Core data provider for SQL Server, as shown in the following markup:
   ```xml
   <Project Sdk="Microsoft.NET.Sdk">
        <PropertyGroup>
            // ...existing code...
            <TreatWarningsAsErrors>true</TreatWarningsAsErrors>
            // ...existing code...
        </PropertyGroup>
    <ItemGroup>
        <ProjectReference Include=
            "..\Northwind.EntityModels\Northwind.EntityModels.csproj" />
    </ItemGroup>
   </Project>
   ```
   cli:
   ```bash
   dotnet add ./Northwind.DataContext/Northwind.DataContext.csproj reference ./Northwind.EntityModels/Northwind.EntityModels.csproj
   dotnet add package Microsoft.EntityFrameworkCore.SqlServer --version 8.0.20 
   ```
3. Delete the `Class1.cs` file.
4. Build the `Northwind.DataContext` project.
5. Move the `NorthwindContext.cs` file from the `Northwind.EntityModels` project/folder to the `Northwind.DataContext` project/folder.
6. In the `Northwind.DataContext` project, add a class named `NorthwindContextExtensions.cs`, and modify its contents to define an extension method that adds the Northwind database context to a collection of dependency services, as shown in the following code:
    ```csharp
    using Microsoft.Data.SqlClient; // SqlConnectionStringBuilder
    using Microsoft.EntityFrameworkCore; // UseSqlServer
    using Microsoft.Extensions.DependencyInjection; // IServiceCollection
    namespace Northwind.EntityModels;
    public static class NorthwindContextExtensions
    {
    /// <summary>
    /// Adds NorthwindContext to the specified IServiceCollection. Uses the SqlServer database provider.
    /// </summary>
    /// <param name="services">The service collection.</param>
    /// <param name="connectionString">Set to override the default.</param>
    /// <returns>An IServiceCollection that can be used to add more services.</returns>
    public static IServiceCollection AddNorthwindContext(
        this IServiceCollection services,
        string? connectionString = null)
    {
        if (connectionString == null)
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
        connectionString = builder.ConnectionString;
      }
      services.AddDbContext<NorthwindContext>(options =>
      {
        options.UseSqlServer(connectionString);
        // Log to console when executing EF Core commands.
        options.LogTo(Console.WriteLine,
          new[] { Microsoft.EntityFrameworkCore
            .Diagnostics.RelationalEventId.CommandExecuting });
      },
      // Register with a transient lifetime to avoid concurrency
      // issues with Blazor Server projects.
      contextLifetime: ServiceLifetime.Transient,
      optionsLifetime: ServiceLifetime.Transient);
      return services;
     }
    }
    ```
7. Build the two class libraries and fix any compiler errors.
