# Step 1: Configure your development environment

In this step, install .NET 10 and confirm that your command line can run a
single C# file directly. EF Core ships as NuGet packages, so no extra global
tool install is required for this demo.

## Install .NET 10 or later

1. Go to [Download .NET 10](https://get.dot.net).
1. Download the **SDK** installer for your operating system.
1. Run the installer and follow the prompts.

### Verify the installed SDK

```bash
dotnet --version
```

You should see `10.0.x` or later.

## Next step

[Step 2: Create your Azure SQL Database](step-2.md)
