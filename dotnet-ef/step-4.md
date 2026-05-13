# Step 4: Insert, update, and delete rows

In this step, replace the read-only code from Step 3 with a complete CRUD demo
that uses Entity Framework Core change tracking and `SaveChanges`. The script
inserts one temporary product, reads it back, updates it, reads it again,
deletes it, and verifies that it is gone.

## Replace the code

Replace the contents of `demo.cs` with this version. Use the same connection
values from Step 3.

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

string productNumber = $"DEMO-{Guid.NewGuid():N}"[..25];

var connectionString = BuildConnectionString(server, database, user, password);

Console.WriteLine("Opening connection...");
using var db = new AdventureWorksContext(connectionString);
db.Database.OpenConnection();

Console.WriteLine("Inserting product...");
var newProduct = new Product
{
    Name = "Blue Bicycle",
    ProductNumber = productNumber,
    StandardCost = 0m,
    ListPrice = 0m,
    SellStartDate = DateTime.UtcNow
};
db.Products.Add(newProduct);
db.SaveChanges();
Console.WriteLine($"Inserted ProductID = {newProduct.ProductID}.");

Console.WriteLine("Selecting products...");
PrintProducts(db);

Console.WriteLine("Updating product...");
var toUpdate = db.Products.Single(p => p.ProductID == newProduct.ProductID);
toUpdate.Name = "Red Bicycle";
var updated = db.SaveChanges();
Console.WriteLine($"Rows updated = {updated}.");

Console.WriteLine("Selecting products...");
PrintProducts(db);

Console.WriteLine("Deleting product...");
var toDelete = db.Products.Single(p => p.ProductID == newProduct.ProductID);
db.Products.Remove(toDelete);
var deleted = db.SaveChanges();
Console.WriteLine($"Rows deleted = {deleted}.");

Console.WriteLine("Selecting products...");
PrintProducts(db);

Console.WriteLine("Done.");

static void PrintProducts(AdventureWorksContext db)
{
    var rows = db.Products
        .AsNoTracking()
        .OrderByDescending(p => p.SellStartDate)
        .Take(5)
        .ToList();

    foreach (var product in rows)
    {
        Console.WriteLine(product);
    }
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

`db.Products.Add(...)` registers the new entity with the EF Core change tracker
in the `Added` state. `db.SaveChanges()` translates that into an `INSERT`
statement and reads the generated `ProductID` back into the entity in the same
round trip:

```csharp
var newProduct = new Product
{
    Name = "Blue Bicycle",
    ProductNumber = productNumber,
    StandardCost = 0m,
    ListPrice = 0m,
    SellStartDate = DateTime.UtcNow
};
db.Products.Add(newProduct);
db.SaveChanges();
Console.WriteLine($"Inserted ProductID = {newProduct.ProductID}.");
```

The product number starts with `DEMO-` and includes random characters so
repeated runs do not collide with earlier runs.

### Parameters: pass values safely

EF Core always sends values as parameters — you never concatenate user input
into SQL. Assigning a property value like `toUpdate.Name = "Red Bicycle"`
becomes a parameter in the generated `UPDATE` statement. This protects against
SQL injection without any extra work in your code.

## UPDATE: modify the row you inserted

Load the entity, change a property, and call `SaveChanges`. EF Core's change
tracker detects which columns changed and generates the `UPDATE` statement
automatically:

```csharp
var toUpdate = db.Products.Single(p => p.ProductID == newProduct.ProductID);
toUpdate.Name = "Red Bicycle";
var updated = db.SaveChanges();
```

`SaveChanges()` returns the number of rows affected. A result of `1` means one
row was updated.

## DELETE: clean up the row

`Remove` marks the entity as `Deleted` in the change tracker, and the next
`SaveChanges` issues the `DELETE`:

```csharp
var toDelete = db.Products.Single(p => p.ProductID == newProduct.ProductID);
db.Products.Remove(toDelete);
var deleted = db.SaveChanges();
```

Because the script deletes only the ID returned by its own `Add` + `SaveChanges`
call, it does not delete existing AdventureWorksLT sample rows.

If the script stops after INSERT but before DELETE, you can clean up demo rows
with this SQL statement:

```sql
DELETE FROM SalesLT.Product
WHERE ProductNumber LIKE 'DEMO-%';
```

## What to explore next

- Try [Azure SQL passwordless authentication](https://learn.microsoft.com/en-us/azure/azure-sql/database/passwordless-overview)
  so you do not store a password in code.
- Use `dotnet ef dbcontext scaffold` to reverse-engineer a `DbContext` from your
  existing Azure SQL schema instead of writing the entity classes by hand.
