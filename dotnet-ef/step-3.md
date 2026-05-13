# Step 3: Connect with EF Core and select rows

In this step, create a single-file C# app that connects to Azure SQL Database
through **Entity Framework Core** and runs a read-only LINQ query against
`SalesLT.Product`.

## Create the file

Create a file named `demo.cs`. Replace the four placeholder values with the
connection values you collected in Step 2.

```csharp
#:package Microsoft.EntityFrameworkCore.SqlServer@9.0.0
#:property PublishAot=false

using System;
using System.Linq;
using Microsoft.EntityFrameworkCore;
using Microsoft.Data.SqlClient;

string server   = "YOUR_SERVER_NAME_HERE.database.windows.net";
string database = "YOUR_DATABASE_NAME_HERE";
string user     = "YOUR_LOGIN_NAME_HERE";
string password = "YOUR_PASSWORD_HERE";

var connectionString = BuildConnectionString(server, database, user, password);

Console.WriteLine("Opening connection...");
using var db = new AdventureWorksContext(connectionString);
db.Database.OpenConnection();
Console.WriteLine("Connected successfully.");

Console.WriteLine("Selecting products...");
var products = db.Products
    .OrderByDescending(p => p.SellStartDate)
    .Take(5)
    .ToList();

foreach (var product in products)
{
    Console.WriteLine(product);
}

string BuildConnectionString(string server, string database, string user, string password)
{
    var builder = new SqlConnectionStringBuilder
    {
        DataSource = $"{server},1433",
        InitialCatalog = database,
        UserID = user,
        Password = password,
        Encrypt = true,
        TrustServerCertificate = false,
        ConnectTimeout = 30,
        ConnectRetryCount = 3,
        ConnectRetryInterval = 10
    };
    return builder.ConnectionString;
}

class AdventureWorksContext : DbContext
{
    private readonly string _connectionString;

    public AdventureWorksContext(string connectionString)
    {
        _connectionString = connectionString;
    }

    public DbSet<Product> Products => Set<Product>();

    protected override void OnConfiguring(DbContextOptionsBuilder options)
    {
        options.UseSqlServer(_connectionString, sql =>
        {
            sql.EnableRetryOnFailure(
                maxRetryCount: 3,
                maxRetryDelay: TimeSpan.FromSeconds(30),
                errorNumbersToAdd: null);
        });
    }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Product>(entity =>
        {
            entity.ToTable("Product", schema: "SalesLT");
            entity.HasKey(p => p.ProductID);
        });
    }
}

class Product
{
    public int ProductID { get; set; }
    public string Name { get; set; } = "";
    public string ProductNumber { get; set; } = "";
    public decimal StandardCost { get; set; }
    public decimal ListPrice { get; set; }
    public DateTime SellStartDate { get; set; }

    public override string ToString() =>
        $"Product {{ ProductID = {ProductID}, Name = {Name}, ProductNumber = {ProductNumber}, ListPrice = {ListPrice} }}";
}
```

## Run the code

```bash
dotnet run demo.cs
```

Expected output looks similar to this. Your product IDs and product names might
be different.

```output
Opening connection...
Connected successfully.
Selecting products...
Product { ProductID = ..., Name = ..., ProductNumber = ..., ListPrice = ... }
Product { ProductID = ..., Name = ..., ProductNumber = ..., ListPrice = ... }
...
```

## How the connection works

### File-based program directives

The two lines at the top are `.NET 10` file-based program directives. They tell
`dotnet run` which NuGet package to restore and configure the project before
compilation:

```csharp
#:package Microsoft.EntityFrameworkCore.SqlServer@9.0.0
#:property PublishAot=false
```

`#:package` references the EF Core SQL Server provider, which pulls in
`Microsoft.EntityFrameworkCore` and `Microsoft.Data.SqlClient` as
dependencies. `#:property PublishAot=false` disables Native AOT publishing,
which EF Core's runtime model building is not compatible with.

## How the connection works

EF Core uses the standard `Microsoft.Data.SqlClient` connection string under the
covers. `BuildConnectionString` builds it from the four values you provided:

```csharp
var builder = new SqlConnectionStringBuilder
{
    DataSource = $"{server},1433",
    InitialCatalog = database,
    UserID = user,
    Password = password,
    Encrypt = true,
    TrustServerCertificate = false,
    ConnectTimeout = 30,
    ConnectRetryCount = 3,
    ConnectRetryInterval = 10
};
```

### Key settings

| Setting | Why it matters |
|---------|----------------|
| `DataSource` | Uses the Azure SQL server name and port `1433` |
| `InitialCatalog` | Connects directly to your database instead of `master` |
| `Encrypt` | Encrypts traffic to Azure SQL Database |
| `TrustServerCertificate` | Validates the server certificate when set to `false` |
| `ConnectTimeout` | Uses a 30-second timeout, which is recommended for internet-based connections |
| `ConnectRetryCount` and `ConnectRetryInterval` | Retry opening idle or broken connections during transient connectivity issues |

### Retry logic

`EnableRetryOnFailure` is the EF Core equivalent of the `SqlClient` retry
provider. EF Core wraps queries and `SaveChanges` calls in an execution
strategy that retries transient failures, such as brief network interruptions
or Azure SQL failover events. Retry logic does not fix persistent configuration
problems, such as an incorrect password, wrong database name, or missing
firewall rule.

## How the LINQ query works

The `Products` `DbSet` is the EF Core entry point for the `SalesLT.Product`
table. The mapping is declared in `OnModelCreating`:

```csharp
modelBuilder.Entity<Product>(entity =>
{
    entity.ToTable("Product", schema: "SalesLT");
    entity.HasKey(p => p.ProductID);
});
```

The LINQ query reads the five most recently added products:

```csharp
var products = db.Products
    .OrderByDescending(p => p.SellStartDate)
    .Take(5)
    .ToList();
```

EF Core translates this into the equivalent T-SQL:

```sql
SELECT TOP(5)
    [p].[ProductID], [p].[Name], [p].[ProductNumber],
    [p].[StandardCost], [p].[ListPrice], [p].[SellStartDate]
FROM [SalesLT].[Product] AS [p]
ORDER BY [p].[SellStartDate] DESC;
```

`ToList()` materializes the rows into `Product` objects.

## Troubleshoot connection issues

If the app cannot connect:

- Confirm the server name ends with `.database.windows.net` and does not include
  `https://`.
- Confirm the database name matches the database you created.
- Confirm your SQL login and password are correct.
- Confirm your current client IP address is allowed in the Azure SQL firewall.
- Confirm outbound TCP traffic on port `1433` is allowed from your network.

## Next step

[Step 4: Insert, update, and delete rows](step-4.md)
