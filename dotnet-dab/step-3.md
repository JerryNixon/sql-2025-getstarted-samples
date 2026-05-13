# Step 3: Initialize Data API builder and start the REST API

In this step, create a DAB configuration for Azure SQL, map one REST entity to
`SalesLT.Product`, and start DAB locally.

## Create the DAB config

Run these commands in an empty local folder where you want to keep your demo
files.

```bash
dab init --database-type mssql --connection-string "Server=tcp:YOUR_SERVER_NAME_HERE.database.windows.net,1433;Initial Catalog=YOUR_DATABASE_NAME_HERE;User ID=YOUR_LOGIN_NAME_HERE;Password=YOUR_PASSWORD_HERE;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;"
```

Add the `SalesLT.Product` table as a REST entity named `Product` with full CRUD
permissions for this local quickstart:

```bash
dab add Product --source "SalesLT.Product" --permissions "anonymous:*"
```

This creates and updates `dab-config.json`.

## Start DAB

```bash
dab start
```

By default, the local API runs at:

- `http://localhost:5000/api`

You can validate that the API can read rows with:

- `GET http://localhost:5000/api/Product`

> [!NOTE]
> Swagger/OpenAPI is available in DAB environments where it is enabled. Nice for
> quick endpoint inspection, but this quickstart stays focused on CLI + C# calls.

## (Optional) quick C# read-only smoke test

Create a file named `demo.cs` and run it while `dab start` is running.

```csharp
using System;
using System.Net.Http;
using System.Threading.Tasks;

using var http = new HttpClient();
var json = await http.GetStringAsync("http://localhost:5000/api/Product");
Console.WriteLine(json);
```

Run:

```bash
dotnet run demo.cs
```

## Next step

[Step 4: Run CRUD from C# with HttpClient](step-4.md)
