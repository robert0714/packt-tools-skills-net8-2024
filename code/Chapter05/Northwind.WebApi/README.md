# Upgrade OpenTelemetry  to new Version
* CLI update 
  * https://www.nuget.org/packages/OpenTelemetry/1.11.2#readme-body-tab
  * https://www.nuget.org/packages/OpenTelemetry.Exporter.Console/1.11.2#readme-body-tab
  * https://www.nuget.org/packages/OpenTelemetry.Extensions.Hosting/1.11.2#readme-body-tab
  * https://www.nuget.org/packages/OpenTelemetry.Instrumentation.AspNetCore/1.11.1#readme-body-tab
  * https://www.nuget.org/packages/OpenTelemetry.Instrumentation.EntityFrameworkCore/1.11.0-beta.2#readme-body-tab 
  * https://www.nuget.org/packages/OpenTelemetry.Instrumentation.SqlClient/1.11.0-beta.2#readme-body-tab 
  ```bash 
  dotnet add package OpenTelemetry.Exporter.Console --version 1.11.2
  dotnet add package OpenTelemetry.Extensions.Hosting --version 1.11.2
  dotnet add package OpenTelemetry.Instrumentation.AspNetCore --version 1.11.*
  dotnet add package OpenTelemetry.Instrumentation.EntityFrameworkCore --version 1.11.0-*
  dotnet add package OpenTelemetry.Instrumentation.SqlClient --version 1.11.0-*
  ```
* In `Northwind.WebApi.csproj` , updated the version of `<PackageReference>`
  ```xml
    // ...existing code...
    { changed code }
    <ItemGroup>
      <PackageReference Include="OpenTelemetry.Exporter.Console" Version="1.11.2" />
      <PackageReference Include="OpenTelemetry.Extensions.Hosting" Version="1.11.2" />
      <PackageReference Include="OpenTelemetry.Instrumentation.AspNetCore" Version="1.11.*" />
      <PackageReference Include="OpenTelemetry.Instrumentation.EntityFrameworkCore" Version="1.11.0-*" />
      <PackageReference Include="OpenTelemetry.Instrumentation.SqlClient" Version="1.11.0-*" />
        ...
    </ItemGroup>
    { changed code }
    // ...existing code...
  ```
  After upgrading , executing the below cli:
  ```bash
  dotnet restore
  dotnet build
  dotnet run
  ```

# Swagger
* https://localhost:50051/swagger/index.html
* http://localhost:50052/swagger/index.html  
* http://localhost:50052/api/products/outofstock

# Development environment
* Startup Container
  ```bash
  docker compose pull
  docker compose up -d
  docker logs -f mssql
  ```
* Add packages
  ```bash
  dotnet add package Microsoft.EntityFrameworkCore.Design --version 8.0.4
  dotnet restore
  dotnet build
  ```  
* [Setting up a database and projects for this book](../../Chapter01/README.md) or Using `dotnet-ef CLI ` for DB migration 
  ```bash
  # Inspect dotnet-ef  
  dotnet tool list -g
  dotnet ef --info

  # Install dotnet-ef
  dotnet tool install --global dotnet-ef

  # Install Schema
  # 在整合終端設定 environment，讓 appsettings.Development.json 被使用，並覆寫 ConnectionStrings
  $env:ASPNETCORE_ENVIRONMENT = 'Development'
  $env:ConnectionStrings__Northwind = 'Server=localhost,1433;Database=Northwind;User Id=sa;Password=StrongP@ssw0rd123!;TrustServerCertificate=True;MultipleActiveResultSets=True;'
  $env:ConnectionStrings__DefaultConnection = 'DataSource=app.db;Cache=Shared'
  #執行 migration（指定 startup project、context，並加 --verbose 查看實際使用的連線）：
  dotnet ef database update --project ..\..\Chapter01\Northwind.EntityModels\ --startup-project . --context NorthwindContext --verbose
  #替代（直接傳入連線字串，不建議在共用環境暴露密碼）
  dotnet ef database update --project ..\..\Chapter01\Northwind.EntityModels\ --startup-project . --connection "Server=localhost,1433;Database=Northwind;User Id=sa;Password=StrongP@ssw0rd123!;TrustServerCertificate=True;" --context NorthwindContext --verbose
  ```
* Modify Config:
  ```json
  {
    "Logging": {
      "LogLevel": {
        "Default": "Information",
        "Microsoft.AspNetCore": "Warning"
      }
    },
    "AllowedHosts": "*"
  }

  ```
  to
  ```json
  { 
    "ConnectionStrings": {
      "Northwind": "Server=localhost,1433;Database=Northwind;User Id=sa;Password=StrongP@ssw0rd123!;TrustServerCertificate=True;MultipleActiveResultSets=True;"
    },
    "Logging": {
      "LogLevel": {
        "Default": "Information",
        "Microsoft.AspNetCore": "Warning"
      }
    },
    "AllowedHosts": "*"
  }
  ```

* [SqlConnectionString key code](../../Chapter01/Northwind.DataContext/NorthwindContext.cs?plain=1#L74-L102)
  ```csharp
  $env:MY_SQL_USR = 'sa'
  $env:MY_SQL_PWD = 'StrongP@ssw0rd123!'
  ```