# Setting up a database and projects for this book
We need a database and some projects that we can use throughout this book. To make it reasonably realistic, we need multiple projects that use common features like a SQL Server database, class libraries, unit tests, and so on.

We will define an entity data model as a pair of reusable class libraries. One part of the pair will define the entities like `Product` and `Customer`. The second part of the pair will define the tables in the database, the default configuration for how to connect to the database, and use the Fluent API to configure additional options for the model.

We will create three projects:
* A class library for entity models like `Category` and `Product` named `Northwind.EntityModels`
* A class library for an EF Core data context named `Northwind.DataContext`
* An xUnit project for unit and integration tests named `Northwind.Tests`

### Using a sample relational database
It would be useful to have a sample database that has a medium complexity and a decent number of sample records. Microsoft offers several sample databases, most of which are too complex for our needs, so instead, we will use a database that was first created in the early 1990s known as **Northwind**.
 
### Setting up SQL Server and the Northwind database
Microsoft offers various editions of its popular and capable SQL Server product for Windows, Linux, and Docker containers.
* To install SQL Server locally on Windows, please see the online instructions at the following [link](../../docs/sql-server/README.md) .
* If you prefer to install SQL Server locally on Linux, then you will find instructions at the following [link](https://learn.microsoft.com/en-us/sql/linux/sql-server-linux-setup?view=sql-server-ver17) .
* If you do not have a Windows computer or if you want to use a cross-platform database system, then please see the online-only section, *Installing Azure SQL Edge in Docker*, found at the following [link](../../docs/sql-server/edge.md) .

### Creating the Northwind database for a local SQL Server
To run a SQL script to create the Northwind sample database for a local SQL Server:
1. Copy the script to create the Northwind database for SQL Server from the following path in your local Git repository: [`/scripts/sql-scripts/Northwind4SQLServer.sql`](../../scripts/sql-scripts/Northwind4SQLServer.sql) into a working folder.
2. Start **SQL Server Management Studio**.
3. In the **Connect** to Server dialog, for Server name, enter . (a dot), meaning the local computer name, and then click the Connect button. If you had to create a named instance, like tools-skills-net8, then enter .\tools-skills-net8.
4. Navigate to **File** | **Open** | **File...**.
5. Browse to select the `Northwind4SQLServer.sql` file and then click the **Open** button.
6. In the toolbar, click **Execute**, and note the **Command(s) completed successfully** message.
7. In **Object Explorer**, expand the **Northwind** database, and then expand **Tables**.
8. Right-click **Products**, click **Select Top 1000 Rows**, and note the returned results.
9. Exit **SQL Server Management Studio**.


# Commands to create the solution
The following commands create the test solution on a Windows machine. For macOS and Unix, update the `ren` command to the OS version of ren to rename a file:
```bash
dotnet new sln -o Chapter01
cd Chapter01
dotnet new classlib -o Northwind.EntityModels
rm .\Northwind.EntityModels\Class1.cs
cd Northwind.EntityModels
dotnet add package Microsoft.EntityFrameworkCore.SqlServer --version 8.0.20
dotnet add package Microsoft.EntityFrameworkCore.Design --version 8.0.20

# generate entity class models for all tables
dotnet ef dbcontext scaffold "Server=127.0.0.1,1433;Database=Northwind;User Id=sa;Password=StrongP@ssw0rd123!;TrustServerCertificate=True;MultipleActiveResultSets=True;" Microsoft.EntityFrameworkCore.SqlServer --namespace Northwind.EntityModels --data-annotations

cd ..
dotnet sln add ./Northwind.EntityModels/Northwind.EntityModels.csproj
dotnet new classlib -o Northwind.DataContext
rm .\Northwind.DataContext\Class1.cs
dotnet add ./Northwind.DataContext/Northwind.DataContext.csproj reference ./Northwind.EntityModels/Northwind.EntityModels.csproj
cd Northwind.DataContext
dotnet add package Microsoft.EntityFrameworkCore.SqlServer --version 8.0.20
cd ..
dotnet sln add ./Northwind.DataContext/Northwind.DataContext.csproj
dotnet new xunit -o Northwind.Tests
dotnet add ./Northwind.Tests/Northwind.Tests.csproj reference ./Northwind.DataContext/Northwind.DataContext.csproj
cd ..
dotnet sln add ./Northwind.Tests/Northwind.Tests.csproj
```
## Creating a class library for entity models using SQL Server
[See Northwind.EntityModels/README.md](Northwind.EntityModels/README.md) 
## Creating a class library for the data context using SQL Server
[See Northwind.DataContext/README.md](Northwind.DataContext/README.md) 
## Creating a test project to check the integration of the class libraries
[See Northwind.Tests/README.md](Northwind.Tests/README.md) 