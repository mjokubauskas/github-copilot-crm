## Installation

Add the following lines to install the Console and MSSQL sinks:
```bash
Install-Package Serilog.Sinks.Console
Install-Package Serilog.Sinks.MSSqlServer
```

## Basic Configuration

### Setup Serilog for Console and MSSQL

```csharp
using Serilog;

public class Program
{
    public static void Main(string[] args)
    {
        Log.Logger = new LoggerConfiguration()
            .MinimumLevel.Information()
            .WriteTo.Console()
            .WriteTo.MSSqlServer(
                connectionString: "YourConnectionString",
                sinkOptions: new MSSqlServerSinkOptions
                {
                    TableName = "Logs",
                    AutoCreateSqlTable = true
                })
            .CreateLogger();

        try
        {
            Log.Information("Starting the application with Logger...");
            CreateHostBuilder(args).Build().Run();
        }
        catch (Exception ex)
        {
            Log.Fatal(ex, "Application startup failed.");
        }
        finally
        {
            Log.CloseAndFlush();
        }
    }

    public static IHostBuilder CreateHostBuilder(string[] args) => Host.CreateDefaultBuilder(args)
        .UseSerilog()
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.UseStartup<Startup>();
        });
}
```

## Advanced Configuration

### Structured Logging

Structured logging allows capturing data in a structured format instead of plain text, which enables better querying and visualization in various platforms like MSSQL, Kibana, or Seq. For example:

```csharp
Log.ForContext("UserId", user.Id)
   .Information("User successfully logged in");
```

### Additional Enrichers

Enrichers allow you to add contextual information to your logs, making it easier to analyze them later. Example:

```csharp
.Enrich.WithMachineName()
.Enrich.WithThreadId()
```
- [Explore Serilog enrichers](https://github.com/serilog/serilog/wiki/Provided-Sinks).