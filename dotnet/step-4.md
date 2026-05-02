# Step 4: Insert, update, and delete rows

In this step, replace the read-only code from Step 3 with a complete CRUD demo.
The script inserts one temporary product, reads it back, updates it, reads it
again, deletes it, and verifies that it is gone.

## Replace the code

Replace the contents of `demo.cs` with this version. Use the same connection
values from Step 3.

```csharp
#r "nuget: Microsoft.Data.SqlClient, 7.0.0"

using System;
using System.Collections.Generic;
using System.Data;
using Microsoft.Data.SqlClient;

string server   = "YOUR_SERVER_NAME_HERE.database.windows.net";
string database = "YOUR_DATABASE_NAME_HERE";
string user     = "YOUR_LOGIN_NAME_HERE";
string password = "YOUR_PASSWORD_HERE";

string productNumber = $"DEMO-{Guid.NewGuid():N}"[..25];

Console.WriteLine("Opening connection...");
using var connection = OpenConnection(server, database, user, password);

Console.WriteLine("Inserting product...");
var productId = InsertProduct(
    connection,
    name: "Blue Bicycle",
    productNumber: productNumber,
    standardCost: 0m,
    listPrice: 0m);
Console.WriteLine($"Inserted ProductID = {productId}.");

Console.WriteLine("Selecting products...");
foreach (var product in SelectProducts(connection))
{
    Console.WriteLine(product);
}

Console.WriteLine("Updating product...");
var updated = UpdateProductName(connection, productId, "Red Bicycle");
Console.WriteLine($"Rows updated = {updated}.");

Console.WriteLine("Selecting products...");
foreach (var product in SelectProducts(connection))
{
    Console.WriteLine(product);
}

Console.WriteLine("Deleting product...");
var deleted = DeleteProduct(connection, productId);
Console.WriteLine($"Rows deleted = {deleted}.");

Console.WriteLine("Selecting products...");
foreach (var product in SelectProducts(connection))
{
    Console.WriteLine(product);
}

Console.WriteLine("Done.");

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

int InsertProduct(
    SqlConnection connection,
    string name,
    string productNumber,
    decimal standardCost,
    decimal listPrice)
{
    var sql = """
        INSERT INTO SalesLT.Product
            (Name, ProductNumber, StandardCost, ListPrice, SellStartDate)
        OUTPUT INSERTED.ProductID
        VALUES
            (@Name, @ProductNumber, @StandardCost, @ListPrice, CURRENT_TIMESTAMP);
        """;

    using var command = new SqlCommand(sql, connection);

    var nameParam = new SqlParameter("@Name", SqlDbType.NVarChar, 50);
    nameParam.Value = name;
    command.Parameters.Add(nameParam);

    var numberParam = new SqlParameter("@ProductNumber", SqlDbType.NVarChar, 25);
    numberParam.Value = productNumber;
    command.Parameters.Add(numberParam);

    var costParam = new SqlParameter("@StandardCost", SqlDbType.Money);
    costParam.Value = standardCost;
    command.Parameters.Add(costParam);

    var priceParam = new SqlParameter("@ListPrice", SqlDbType.Money);
    priceParam.Value = listPrice;
    command.Parameters.Add(priceParam);

    return (int)command.ExecuteScalar();
}

int UpdateProductName(SqlConnection connection, int productId, string name)
{
    var sql = """
        UPDATE SalesLT.Product
        SET
            Name = @Name
        WHERE
            ProductID = @ProductID;
        """;

    using var command = new SqlCommand(sql, connection);

    var nameParam = new SqlParameter("@Name", SqlDbType.NVarChar, 50);
    nameParam.Value = name;
    command.Parameters.Add(nameParam);

    var idParam = new SqlParameter("@ProductID", SqlDbType.Int);
    idParam.Value = productId;
    command.Parameters.Add(idParam);

    return command.ExecuteNonQuery();
}

int DeleteProduct(SqlConnection connection, int productId)
{
    var sql = """
        DELETE FROM SalesLT.Product
        WHERE
            ProductID = @ProductID;
        """;

    using var command = new SqlCommand(sql, connection);

    var idParam = new SqlParameter("@ProductID", SqlDbType.Int);
    idParam.Value = productId;
    command.Parameters.Add(idParam);

    return command.ExecuteNonQuery();
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

Expected output looks similar to this. Your product ID and sample rows might be
different.

```output
Opening connection...
Inserting product...
Inserted ProductID = ...
Selecting products...
Product { ProductID = ..., Name = Blue Bicycle, ProductNumber = DEMO-..., ListPrice = 0.0000 }
...
Updating product...
Rows updated = 1.
Selecting products...
Product { ProductID = ..., Name = Red Bicycle, ProductNumber = DEMO-..., ListPrice = 0.0000 }
...
Deleting product...
Rows deleted = 1.
Selecting products...
...
Done.
```

## INSERT: create one temporary row

`InsertProduct` inserts one row into `SalesLT.Product`. The product number starts
with `DEMO-` and includes random characters so repeated runs do not collide with
earlier runs.

The SQL uses `OUTPUT INSERTED.ProductID` to return the new primary key in the
same round trip:

```sql
INSERT INTO SalesLT.Product
    (Name, ProductNumber, StandardCost, ListPrice, SellStartDate)
OUTPUT INSERTED.ProductID
VALUES
    (@Name, @ProductNumber, @StandardCost, @ListPrice, CURRENT_TIMESTAMP);
```

`ExecuteScalar()` reads that single returned value.

### Parameters: pass values safely

Every write operation uses `SqlParameter` values instead of string
interpolation. Parameters keep SQL text separate from data values and help
protect against SQL injection.

```csharp
var nameParam = new SqlParameter("@Name", SqlDbType.NVarChar, 50);
nameParam.Value = name;
command.Parameters.Add(nameParam);
```

The parameter type and length should match the database column. For example,
`Name` is `nvarchar(50)`, and `StandardCost` and `ListPrice` use `money`.

## UPDATE: modify the row you inserted

`UpdateProductName` updates one row by `ProductID`:

```sql
UPDATE SalesLT.Product
SET
    Name = @Name
WHERE
    ProductID = @ProductID;
```

`ExecuteNonQuery()` returns the number of rows affected. A result of `1` means
one row was updated.

## DELETE: clean up the row

`DeleteProduct` removes the same row by `ProductID`:

```sql
DELETE FROM SalesLT.Product
WHERE
    ProductID = @ProductID;
```

Because the script deletes only the ID returned by its own INSERT statement, it
does not delete existing AdventureWorksLT sample rows.

If the script stops after INSERT but before DELETE, you can clean up demo rows
with this SQL statement:

```sql
DELETE FROM SalesLT.Product
WHERE ProductNumber LIKE 'DEMO-%';
```

## What to explore next

- Try [Azure SQL passwordless authentication](https://learn.microsoft.com/en-us/azure/azure-sql/database/passwordless-overview)
  so you do not store a password in code.
