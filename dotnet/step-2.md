# Step 2: Create your Azure SQL Database

The demo uses the `SalesLT.Product` table from the **AdventureWorksLT** sample
database. In this step, create an Azure SQL Database with sample data, allow
your local computer to connect, and collect the four values the code needs.

> [!NOTE]
> For the complete Azure portal walkthrough, see [Quickstart: Create a single database - Azure SQL Database](https://learn.microsoft.com/en-us/azure/azure-sql/database/single-database-create-quickstart?view=azuresql&tabs=azure-portal). The steps below are the shortened version tailored to this .NET demo.

## Create the database

1. Open the Azure SQL Database create page in the [Azure portal](https://portal.azure.com/#servicemenu/SqlAzureExtension/AzureSqlHub/SingleDatabase).
2. Choose either **SQL database** or **SQL database (Free Offer)**.

> [!TIP]
> The **SQL database (Free Offer)** option works for this demo, but there is one
> extra click. If the portal shows the **Free offer applied** page, select
> **Advanced configuration** before you select **Review + create**. The simplified
> free-offer page creates a database with defaults, but this demo needs the
> **Sample** data option so the `SalesLT.Product` table exists.
> [Learn more](https://aka.ms/sqlfreeoffer).

3. Move through the wizard. Most values can use your own preferences or the
    portal defaults. For this demo, make sure these values are set:

| Other setting | Demo value |
|-|-|
| Basics → Database name | `AdventureWorksLT` |
| Basics → Authentication method | Use SQL authentication |
| Basics → Server admin login and password | Username/Password |
| Networking → Add current client IP address | `Yes` |
| Additional settings → Use existing data | `Sample` |

4. Select **Review + create**, review the settings, and then select **Create**.

## Collect your connection values

After deployment finishes, open the database in the Azure portal and record
these values. You use them in Step 3 and Step 4.

| Code variable | Value |
|-|-|
| `server` | The server name, such as `myserver.database.windows.net` |
| `database` | The database name, such as `AdventureWorksLT` |
| `user` | The SQL server admin login you created |
| `password` | The SQL server admin password you created |

## Next step

[Step 3: Connect and run your first query](step-3.md)
