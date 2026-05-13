# Prompt: Generate the dotnet-ef getting-started sample

Use this prompt to regenerate or extend the `/dotnet-ef` walkthrough.

---

You are authoring a getting-started walkthrough for **Azure SQL Database** with
**.NET** and **Entity Framework Core**. Follow the same structure and tone as
the existing `/dotnet` and `/dotnet-dab` samples in this repository.

## Output

Produce five Markdown files inside a `/dotnet-ef` folder:

| File | Purpose |
|------|---------|
| `getting-started.md` | Overview, prerequisites, and step table |
| `step-1.md` | Install .NET 10 and verify |
| `step-2.md` | Create Azure SQL Database with AdventureWorksLT sample data |
| `step-3.md` | Connect with EF Core DbContext and run a read-only LINQ SELECT |
| `step-4.md` | Insert, update, and delete one row with `SaveChanges` |

## Rules

- Mirror the voice, heading levels, table formats, and note/tip callout style
  of `/dotnet/step-2.md` through `/dotnet/step-4.md`.
- Use the **same database and table**: `SalesLT.Product` from AdventureWorksLT.
- Use the **same four connection variables**: `server`, `database`, `user`,
  `password`.
- Use the **same demo row**: insert "Blue Bicycle", update to "Red Bicycle",
  delete. Use `DEMO-{Guid}` as the product number.
- Use **`#:package`** and **`#:property`** directives (not `#r "nuget:..."`)
  for .NET 10 file-based program execution:
  ```
  #:package Microsoft.EntityFrameworkCore.SqlServer@9.0.0
  #:property PublishAot=false
  ```
- Do **not** use `dotnet new` or a `.csproj` file. The demo runs with
  `dotnet run demo.cs`.
- Use `Microsoft.EntityFrameworkCore.SqlServer` version `9.0.0`.
- Map `SalesLT.Product` using `OnModelCreating` with `ToTable("Product", "SalesLT")`.
  Do **not** use scaffolding or Migrations.
- Disable `PublishAot` because EF Core runtime model building is incompatible
  with .NET 10's default NativeAOT file-based compilation.
- Use `EnableRetryOnFailure(maxRetryCount: 3, maxRetryDelay: TimeSpan.FromSeconds(30), errorNumbersToAdd: null)`
  in `OnConfiguring` for transient fault handling.
- Use `AsNoTracking()` in the read-within-CRUD `PrintProducts` helper.
- `SaveChanges()` returns the row count — show this in the output.
- After INSERT, `newProduct.ProductID` is populated automatically by EF Core.
- Explain the EF Core change-tracker states (Added → Deleted) in step-4.
- Include a SQL cleanup snippet for runs that stop early:
  ```sql
  DELETE FROM SalesLT.Product WHERE ProductNumber LIKE 'DEMO-%';
  ```
- End step-4 with a "What to explore next" section pointing to passwordless
  auth and `dotnet ef dbcontext scaffold`.

## `Product` entity

```csharp
class Product
{
    public int ProductID { get; set; }
    public string Name { get; set; } = "";
    public string ProductNumber { get; set; } = "";
    public decimal StandardCost { get; set; }
    public decimal ListPrice { get; set; }
    public DateTime SellStartDate { get; set; }
}
```

## `AdventureWorksContext` shape

```csharp
class AdventureWorksContext : DbContext
{
    // constructor takes connection string
    public DbSet<Product> Products => Set<Product>();
    // OnConfiguring: UseSqlServer + EnableRetryOnFailure
    // OnModelCreating: ToTable("Product", "SalesLT") + HasKey(p => p.ProductID)
}
```

## Verified output (step-3)

```output
Opening connection...
Connected successfully.
Selecting products...
Product { ProductID = 864, Name = Classic Vest, S, ProductNumber = VE-C304-S, ListPrice = 63.5000 }
...
```

## Verified output (step-4)

```output
Opening connection...
Inserting product...
Inserted ProductID = 1000.
Selecting products...
Product { ProductID = 1000, Name = Blue Bicycle, ProductNumber = DEMO-..., ListPrice = 0.0000 }
...
Updating product...
Rows updated = 1.
Selecting products...
Product { ProductID = 1000, Name = Red Bicycle, ProductNumber = DEMO-..., ListPrice = 0.0000 }
...
Deleting product...
Rows deleted = 1.
Selecting products...
...
Done.
```
