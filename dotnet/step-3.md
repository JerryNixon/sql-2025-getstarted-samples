# Step 3: Connect to the database and select rows

In this step, create a single-file C# app that connects to Azure SQL Database
and runs a read-only query against `SalesLT.Product`.

## Create the file

Create a file named `demo.cs`. Replace the four placeholder values with the
connection values you collected in Step 2.

```csharp
#r "nuget: Microsoft.Data.SqlClient, 7.0.0"

using System;
using System.Collections.Generic;
using Microsoft.Data.SqlClient;

string server   = "YOUR_SERVER_NAME_HERE.database.windows.net";
string database = "YOUR_DATABASE_NAME_HERE";
string user     = "YOUR_LOGIN_NAME_HERE";
string password = "YOUR_PASSWORD_HERE";

Console.WriteLine("Opening connection...");
using var connection = OpenConnection(server, database, user, password);
Console.WriteLine("Connected successfully.");

Console.WriteLine("Selecting products...");
foreach (var product in SelectProducts(connection))
{
    Console.WriteLine(product);
}

SqlConnection OpenConnection(string server, string database, string user, string password)
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

    var connection = new SqlConnection(builder.ConnectionString);

    var retryProvider = SqlConfigurableRetryFactory.CreateExponentialRetryProvider(
        new SqlRetryLogicOption
        {
            NumberOfTries = 3,
            DeltaTime = TimeSpan.FromSeconds(5),
            MaxTimeInterval = TimeSpan.FromSeconds(30)
        });

    connection.RetryLogicProvider = retryProvider;
    connection.Open();
    return connection;
}

IEnumerable<Product> SelectProducts(SqlConnection connection)
{
    var sql = """
        SELECT
            TOP 5
                ProductID,
                Name,
                ProductNumber,
                ListPrice
        FROM
            SalesLT.Product
        ORDER BY
            SellStartDate DESC;
        """;

    using var command = new SqlCommand(sql, connection);
    using var reader = command.ExecuteReader();

    while (reader.Read())
    {
        yield return new Product(
            reader.GetInt32(0),
            reader.GetString(1),
            reader.GetString(2),
            reader.GetDecimal(3)
        );
    }
}

record Product(int ProductID, string Name, string ProductNumber, decimal ListPrice);
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

`OpenConnection` uses `SqlConnectionStringBuilder` to create a connection string
from the four values you provided:

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

This helps the app recover from transient failures, such as brief network
interruptions or Azure SQL failover events. Retry logic does not fix persistent
configuration problems, such as an incorrect password, wrong database name, or
missing firewall rule.

## How the SELECT query works

`SelectProducts` reads the five most recently added products:

```sql
SELECT
    TOP 5
        ProductID,
        Name,
        ProductNumber,
        ListPrice
FROM
    SalesLT.Product
ORDER BY
    SellStartDate DESC;
```

`SqlDataReader` streams rows one at a time. The method uses `yield return` so
each `Product` is returned as it is read.

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
