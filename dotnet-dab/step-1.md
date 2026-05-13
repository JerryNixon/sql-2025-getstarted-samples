# Step 1: Configure your development environment

In this step, install .NET 10 and Data API builder (DAB), then verify both are
available from your command line.

## Install .NET 10 or later

1. Go to [Download .NET 10](https://get.dot.net).
1. Download the **SDK** installer for your operating system.
1. Run the installer and follow the prompts.

### Verify the installed SDK

```bash
dotnet --version
```

## Install the DAB CLI

Install DAB as a global .NET tool:

```bash
dotnet tool install --global Microsoft.DataApiBuilder
```

If you already installed it, update it:

```bash
dotnet tool update --global Microsoft.DataApiBuilder
```

### Verify DAB

```bash
dab --version
```

## Next step

[Step 2: Create your Azure SQL Database](step-2.md)
