# Step 4: Run CRUD from C# using Data API builder

In this step, replace the read-only sample with a complete CRUD demo.
The script inserts one temporary product through DAB REST, reads rows,
updates the inserted row, reads rows again, deletes the row, and verifies it is
gone.

> [!IMPORTANT]
> Keep `dab start` running while you run this C# file.

## Replace the code

Replace the contents of `demo.cs` with this version.

```csharp
using System;
using System.Collections.Generic;
using System.Net.Http;
using System.Net.Http.Json;
using System.Text.Json;
using System.Text.Json.Serialization;
using System.Threading.Tasks;

var baseUrl = "http://localhost:5000/api";
using var http = new HttpClient { BaseAddress = new Uri(baseUrl + "/") };

string productNumber = $"DEMO-{Guid.NewGuid():N}"[..25];

Console.WriteLine("Inserting product through DAB...");
var createPayload = new
{
    Name = "Blue Bicycle",
    ProductNumber = productNumber,
    StandardCost = 0m,
    ListPrice = 0m,
    SellStartDate = DateTime.UtcNow
};

var createResponse = await http.PostAsJsonAsync("Product", createPayload);
createResponse.EnsureSuccessStatusCode();
var created = await createResponse.Content.ReadFromJsonAsync<Product>(JsonOptions());

if (created is null)
{
    throw new InvalidOperationException("Create response did not return a product body.");
}

Console.WriteLine($"Inserted ProductID = {created.ProductID}.");

Console.WriteLine("Selecting products...");
await PrintProducts(http);

Console.WriteLine("Updating product through DAB...");
var updatePayload = new
{
    Name = "Red Bicycle"
};

var updateResponse = await http.PatchAsJsonAsync($"Product/{created.ProductID}", updatePayload);
updateResponse.EnsureSuccessStatusCode();
Console.WriteLine("Update completed.");

Console.WriteLine("Selecting products...");
await PrintProducts(http);

Console.WriteLine("Deleting product through DAB...");
var deleteResponse = await http.DeleteAsync($"Product/{created.ProductID}");
deleteResponse.EnsureSuccessStatusCode();
Console.WriteLine("Delete completed.");

Console.WriteLine("Selecting products...");
await PrintProducts(http);

Console.WriteLine("Done.");

static async Task PrintProducts(HttpClient http)
{
    using var response = await http.GetAsync("Product");
    response.EnsureSuccessStatusCode();

    var payload = await response.Content.ReadAsStringAsync();
    var page = JsonSerializer.Deserialize<ProductPage>(payload, JsonOptions());

    if (page?.Value is null || page.Value.Count == 0)
    {
        Console.WriteLine("No rows returned.");
        return;
    }

    foreach (var product in page.Value)
    {
        Console.WriteLine(product);
    }
}

static JsonSerializerOptions JsonOptions() => new()
{
    PropertyNameCaseInsensitive = true
};

record Product(
    int ProductID,
    string Name,
    string ProductNumber,
    decimal ListPrice);

record ProductPage(
    [property: JsonPropertyName("value")] List<Product> Value);
```

## Run the code

```bash
dotnet run demo.cs
```

Expected output looks similar to this. Your product ID and sample rows might be
different.

```output
Inserting product through DAB...
Inserted ProductID = ...
Selecting products...
Product { ProductID = ..., Name = Blue Bicycle, ProductNumber = DEMO-..., ListPrice = 0.0000 }
...
Updating product through DAB...
Update completed.
Selecting products...
Product { ProductID = ..., Name = Red Bicycle, ProductNumber = DEMO-..., ListPrice = 0.0000 }
...
Deleting product through DAB...
Delete completed.
Selecting products...
...
Done.
```

## CRUD calls used by the C# code

- **Create**: `POST /api/Product`
- **Read**: `GET /api/Product`
- **Update**: `PATCH /api/Product/{ProductID}`
- **Delete**: `DELETE /api/Product/{ProductID}`

Because the script deletes only the ID returned by its own create call, it does
not delete existing AdventureWorksLT sample rows.

If a run stops after create but before delete, you can clean up demo rows with
this SQL statement in Azure SQL:

```sql
DELETE FROM SalesLT.Product
WHERE ProductNumber LIKE 'DEMO-%';
```

## What to explore next

- Try [Azure SQL passwordless authentication](https://learn.microsoft.com/en-us/azure/azure-sql/database/passwordless-overview)
  so you do not store a password in plain text.
