# Get started with Azure SQL, Data API builder, and .NET

This command-line demo shows how to keep your database in Azure SQL Database and
run your app locally through **Data API builder (DAB)**.

You start with a clean development environment, create a sample database,
install and configure DAB, start a local REST API, and then use a single-file
C# app with `HttpClient` to read, insert, update, and delete one demo row.

## Prerequisites

- An Azure account. [Create one free.](https://azure.microsoft.com/free/)
- .NET 10 SDK. [Download .NET 10](https://dotnet.microsoft.com/download/dotnet/10.0).

## What you'll do

You will work through four short steps:

1. Install the tools you need, including the DAB CLI.
1. Create a database with sample data.
1. Initialize DAB config and start a local REST API over Azure SQL.
1. Use C# `HttpClient` to run CRUD operations through DAB.

The code appears when you need it in Step 3 and Step 4. You don't need to
create a .NET project or solution file.

## Steps

| Step | What you do |
|------|-------------|
| [Step 1](step-1.md) | Install .NET 10, install DAB CLI, and verify both tools |
| [Step 2](step-2.md) | Create an Azure SQL Database with AdventureWorksLT sample data |
| [Step 3](step-3.md) | Initialize DAB, map `SalesLT.Product`, and run the REST API locally |
| [Step 4](step-4.md) | Run CRUD from C# using `HttpClient` against DAB |

Start with [Step 1](step-1.md).
