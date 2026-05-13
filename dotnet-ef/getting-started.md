# Get started with Azure SQL, Entity Framework Core, and .NET

This command-line demo shows how to connect to Azure SQL Database from .NET with
**Entity Framework Core** (EF Core) using the `Microsoft.EntityFrameworkCore.SqlServer`
provider. EF Core talks directly to Azure SQL — there is no middle-tier API.

You start with a clean development environment, create a sample database,
connect to it through a `DbContext`, read rows, and then safely insert,
update, and delete one demo row by calling `SaveChanges`.

## Prerequisites

- An Azure account. [Create one free.](https://azure.microsoft.com/free/)
- .NET 10 SDK. [Download .NET 10](https://dotnet.microsoft.com/download/dotnet/10.0).

## What you'll do

You will work through four short steps:

1. Install the tools you need.
1. Create a database with sample data.
1. Run a read-only LINQ query to prove the connection works.
1. Insert, update, and delete one row that the demo creates.

The code appears when you need it in Step 3 and Step 4. You don't need to create
a .NET project or solution file.

## Steps

| Step | What you do |
|------|-------------|
| [Step 1](step-1.md) | Install .NET 10 and verify single-file C# execution |
| [Step 2](step-2.md) | Create an Azure SQL Database with the AdventureWorksLT sample data |
| [Step 3](step-3.md) | Connect with EF Core and run a read-only LINQ query |
| [Step 4](step-4.md) | Insert, update, and delete one demo row through `SaveChanges` |

Start with [Step 1](step-1.md).
