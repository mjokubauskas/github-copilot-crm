# Setting up Structured Logging with Serilog for Console and Microsoft SQL Server in .NET 6

## Overview of Structured Logging

### What is Structured Logging?
Structured logging is a modern approach to logging where log entries are treated as structured data rather than plain text. Instead of writing unstructured log messages like "User John logged in at 10:30 AM", structured logging captures this information as key-value pairs (e.g., `{ "User": "John", "Action": "Login", "Timestamp": "10:30 AM" }`).

### Benefits of Structured Logging
- **Enhanced Searchability**: Easily query and filter logs based on specific fields and properties
- **Better Analytics**: Analyze patterns and trends across your application using structured data
- **Improved Debugging**: Quickly locate issues by filtering on specific properties like user ID, transaction ID, or error codes
- **Integration-Friendly**: Seamlessly integrate with log aggregation and monitoring tools
- **Context Preservation**: Maintain rich contextual information throughout your application's execution flow

### Why Serilog?
Serilog is the leading structured logging library for .NET applications, offering:
- **Simple API**: Easy-to-use, intuitive logging interface
- **Multiple Sinks**: Write logs to various destinations (console, files, databases, cloud services)
- **Performance**: Highly optimized for minimal overhead
- **Flexibility**: Extensive configuration options and enrichers
- **Community Support**: Large ecosystem with numerous extensions and integrations
- **Native .NET Integration**: First-class support for .NET 6+ and modern .NET features

## Setup Dependencies

To implement structured logging with Serilog for both console and Microsoft SQL Server, you need to install the following NuGet packages:

### Required Packages
1. **Serilog**: Core Serilog library
2. **Serilog.Sinks.Console**: For logging to the console
3. **Serilog.Sinks.MSSqlServer**: For logging to Microsoft SQL Server

### Installation Commands

Using .NET CLI:
```bash
dotnet add package Serilog
dotnet add package Serilog.Sinks.Console
dotnet add package Serilog.Sinks.MSSqlServer
```

Using NuGet Package Manager Console:
```bash
Install-Package Serilog
Install-Package Serilog.Sinks.Console
Install-Package Serilog.Sinks.MSSqlServer
```

### Additional Recommended Packages
For enhanced integration with ASP.NET Core:
```bash
dotnet add package Serilog.AspNetCore
dotnet add package Serilog.Extensions.Hosting
```

## Configure Serilog

### Connection String Setup
Before configuring Serilog, ensure you have a valid connection string to your Microsoft SQL Server database. Add this to your `appsettings.json`:

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=your-server-name;Database=your-database-name;User Id=your-username;Password=your-password;TrustServerCertificate=True;",
    "LoggingConnection": "Server=your-server-name;Database=your-database-name;User Id=your-username;Password=your-password;TrustServerCertificate=True;"
  },
  "Serilog": {
    "MinimumLevel": {
      "Default": "Information",
      "Override": {
        "Microsoft": "Warning",
        "System": "Warning"
      }
    }
  }
}
```

### Configuration in appsettings.json
You can also configure Serilog directly in `appsettings.json` for easier management:

```json
{
  "Serilog": {
    "Using": [ "Serilog.Sinks.Console", "Serilog.Sinks.MSSqlServer" ],
    "MinimumLevel": {
      "Default": "Information",
      "Override": {
        "Microsoft": "Warning",
        "Microsoft.AspNetCore": "Warning",
        "System": "Warning"
      }
    },
    "WriteTo": [
      {
        "Name": "Console",
        "Args": {
          "outputTemplate": "[{Timestamp:HH:mm:ss} {Level:u3}] {Message:lj}{NewLine}{Exception}"
        }
      },
      {
        "Name": "MSSqlServer",
        "Args": {
          "connectionString": "LoggingConnection",
          "tableName": "Logs",
          "autoCreateSqlTable": true,
          "restrictedToMinimumLevel": "Information"
        }
      }
    ],
    "Enrich": [ "FromLogContext", "WithMachineName", "WithThreadId" ]
  }
}
```

## Initializing Serilog in Program.cs

### For .NET 6 Minimal API Applications

Here's how to configure Serilog in a .NET 6 application using the minimal hosting model:

```csharp
using Serilog;
using Serilog.Sinks.MSSqlServer;
using System.Collections.ObjectModel;
using System.Data;

var builder = WebApplication.CreateBuilder(args);

// Configure Serilog
var logger = new LoggerConfiguration()
    .MinimumLevel.Information()
    .MinimumLevel.Override("Microsoft", Serilog.Events.LogEventLevel.Warning)
    .MinimumLevel.Override("System", Serilog.Events.LogEventLevel.Warning)
    .Enrich.FromLogContext()
    .Enrich.WithMachineName()
    .Enrich.WithThreadId()
    .WriteTo.Console(
        outputTemplate: "[{Timestamp:yyyy-MM-dd HH:mm:ss.fff zzz}] [{Level:u3}] {Message:lj}{NewLine}{Exception}")
    .WriteTo.MSSqlServer(
        connectionString: builder.Configuration.GetConnectionString("LoggingConnection"),
        sinkOptions: new MSSqlServerSinkOptions 
        { 
            TableName = "Logs",
            AutoCreateSqlTable = true,
            SchemaName = "dbo"
        },
        restrictedToMinimumLevel: Serilog.Events.LogEventLevel.Information,
        columnOptions: GetSqlColumnOptions())
    .CreateLogger();

// Clear default logging providers and add Serilog
builder.Logging.ClearProviders();
builder.Logging.AddSerilog(logger);

// Set as the global logger (optional but recommended)
Log.Logger = logger;

try
{
    Log.Information("Starting application...");
    
    // Add services to the container
    builder.Services.AddControllers();
    builder.Services.AddEndpointsApiExplorer();
    builder.Services.AddSwaggerGen();

    var app = builder.Build();

    // Configure the HTTP request pipeline
    if (app.Environment.IsDevelopment())
    {
        app.UseSwagger();
        app.UseSwaggerUI();
    }

    app.UseHttpsRedirection();
    app.UseAuthorization();
    app.MapControllers();

    Log.Information("Application started successfully");
    app.Run();
}
catch (Exception ex)
{
    Log.Fatal(ex, "Application terminated unexpectedly");
}
finally
{
    Log.CloseAndFlush();
}

// Helper method to configure SQL column options
static ColumnOptions GetSqlColumnOptions()
{
    var columnOptions = new ColumnOptions();
    
    // Remove default columns that you don't want
    columnOptions.Store.Remove(StandardColumn.Properties);
    columnOptions.Store.Remove(StandardColumn.MessageTemplate);
    
    // Customize standard columns
    columnOptions.TimeStamp.ConvertToUtc = true;
    columnOptions.Level.StoreAsEnum = false;
    
    // Add custom columns
    columnOptions.AdditionalColumns = new Collection<SqlColumn>
    {
        new SqlColumn 
        { 
            ColumnName = "MachineName", 
            DataType = SqlDbType.NVarChar, 
            DataLength = 100, 
            AllowNull = true 
        },
        new SqlColumn 
        { 
            ColumnName = "ThreadId", 
            DataType = SqlDbType.Int, 
            AllowNull = true 
        },
        new SqlColumn 
        { 
            ColumnName = "RequestPath", 
            DataType = SqlDbType.NVarChar, 
            DataLength = 500, 
            AllowNull = true 
        }
    };
    
    return columnOptions;
}
```

### Alternative: Using appsettings.json Configuration

If you prefer to configure Serilog via `appsettings.json`, use this simpler approach:

```csharp
using Serilog;

var builder = WebApplication.CreateBuilder(args);

// Read configuration from appsettings.json
// services parameter enables dependency injection integration for enrichers and sinks
builder.Host.UseSerilog((context, services, configuration) => configuration
    .ReadFrom.Configuration(context.Configuration)
    .ReadFrom.Services(services) // Allows Serilog to access registered services
    .Enrich.FromLogContext());

try
{
    Log.Information("Starting application...");
    
    // Add services
    builder.Services.AddControllers();
    
    var app = builder.Build();
    
    // Configure pipeline
    app.UseHttpsRedirection();
    app.UseAuthorization();
    app.MapControllers();
    
    Log.Information("Application started successfully");
    app.Run();
}
catch (Exception ex)
{
    Log.Fatal(ex, "Application terminated unexpectedly");
}
finally
{
    Log.CloseAndFlush();
}
```

### Logging Usage Examples

Once configured, you can use Serilog throughout your application:

```csharp
// In a controller or service
public class CustomerController : ControllerBase
{
    private readonly ILogger<CustomerController> _logger;

    public CustomerController(ILogger<CustomerController> logger)
    {
        _logger = logger;
    }

    [HttpGet("{id}")]
    public async Task<IActionResult> GetCustomer(int id)
    {
        _logger.LogInformation("Fetching customer with ID: {CustomerId}", id);
        
        try
        {
            var customer = await _customerService.GetByIdAsync(id);
            
            if (customer == null)
            {
                _logger.LogWarning("Customer not found: {CustomerId}", id);
                return NotFound();
            }
            
            _logger.LogInformation("Successfully retrieved customer: {CustomerId}, Name: {CustomerName}", 
                customer.Id, customer.Name);
            return Ok(customer);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error occurred while fetching customer: {CustomerId}", id);
            return StatusCode(500, "Internal server error");
        }
    }
}
```

## Previewing Logs

### Database Table Structure
When Serilog creates the `Logs` table in SQL Server, it will have the following default structure:

| Column | Type | Description |
|--------|------|-------------|
| Id | int (Identity) | Primary key |
| Message | nvarchar(max) | Log message |
| MessageTemplate | nvarchar(max) | Message template |
| Level | nvarchar(128) | Log level (Information, Warning, Error, etc.) |
| TimeStamp | datetime | When the log was created |
| Exception | nvarchar(max) | Exception details (if any) |
| Properties | xml | Additional properties |
| LogEvent | nvarchar(max) | Complete log event |

### SQL Queries for Viewing Logs

#### View All Recent Logs
```sql
SELECT TOP 100 
    Id,
    TimeStamp,
    Level,
    Message,
    Exception
FROM Logs
ORDER BY TimeStamp DESC;
```

#### Filter by Log Level
```sql
-- View only errors
SELECT 
    Id,
    TimeStamp,
    Level,
    Message,
    Exception
FROM Logs
WHERE Level = 'Error'
ORDER BY TimeStamp DESC;

-- View warnings and errors
SELECT 
    Id,
    TimeStamp,
    Level,
    Message,
    Exception
FROM Logs
WHERE Level IN ('Warning', 'Error')
ORDER BY TimeStamp DESC;
```

#### Filter by Time Range
```sql
-- Logs from the last hour
SELECT 
    Id,
    TimeStamp,
    Level,
    Message
FROM Logs
WHERE TimeStamp >= DATEADD(HOUR, -1, GETUTCDATE())
ORDER BY TimeStamp DESC;

-- Logs from today
SELECT 
    Id,
    TimeStamp,
    Level,
    Message
FROM Logs
WHERE CAST(TimeStamp AS DATE) = CAST(GETUTCDATE() AS DATE)
ORDER BY TimeStamp DESC;
```

#### Search for Specific Messages
```sql
-- Find logs containing specific keywords
SELECT 
    Id,
    TimeStamp,
    Level,
    Message,
    Exception
FROM Logs
WHERE Message LIKE '%customer%' 
   OR Message LIKE '%error%'
ORDER BY TimeStamp DESC;
```

#### Error Analysis
```sql
-- Count errors by hour
SELECT 
    DATEPART(HOUR, TimeStamp) AS Hour,
    COUNT(*) AS ErrorCount
FROM Logs
WHERE Level = 'Error'
  AND TimeStamp >= DATEADD(DAY, -1, GETUTCDATE())
GROUP BY DATEPART(HOUR, TimeStamp)
ORDER BY Hour;

-- Most common errors
SELECT 
    LEFT(Message, 100) AS ErrorMessage,
    COUNT(*) AS Occurrences
FROM Logs
WHERE Level = 'Error'
  AND TimeStamp >= DATEADD(DAY, -7, GETUTCDATE())
GROUP BY LEFT(Message, 100)
ORDER BY Occurrences DESC;
```

### Using SQL Server Management Studio (SSMS)
1. Connect to your SQL Server instance
2. Navigate to your database
3. Find the `Logs` table under `Tables`
4. Right-click and select "Select Top 1000 Rows" to view recent logs
5. Modify the query as needed using the examples above

### Creating Views for Common Queries
You can create SQL views for frequently used queries:

```sql
-- Create a view for today's errors
CREATE VIEW vw_TodayErrors AS
SELECT 
    Id,
    TimeStamp,
    Message,
    Exception
FROM Logs
WHERE Level = 'Error'
  AND CAST(TimeStamp AS DATE) = CAST(GETUTCDATE() AS DATE);

-- Query the view
SELECT * FROM vw_TodayErrors ORDER BY TimeStamp DESC;
```

## Advanced Features

### Custom Column Mapping

You can customize the database columns to fit your specific needs:

```csharp
static ColumnOptions GetAdvancedColumnOptions()
{
    var columnOptions = new ColumnOptions();
    
    // Disable automatic column creation for some standard columns
    columnOptions.Store.Remove(StandardColumn.Properties);
    columnOptions.Store.Remove(StandardColumn.MessageTemplate);
    
    // Customize TimeStamp column
    columnOptions.TimeStamp.ConvertToUtc = true;
    columnOptions.TimeStamp.ColumnName = "LogTimestamp";
    
    // Customize Level column
    columnOptions.Level.StoreAsEnum = true;
    columnOptions.Level.ColumnName = "LogLevel";
    
    // Customize Message column
    columnOptions.Message.ColumnName = "LogMessage";
    
    // Add custom columns with specific data types
    columnOptions.AdditionalColumns = new Collection<SqlColumn>
    {
        new SqlColumn 
        { 
            ColumnName = "UserName", 
            DataType = SqlDbType.NVarChar, 
            DataLength = 100,
            AllowNull = true,
            PropertyName = "UserName"
        },
        new SqlColumn 
        { 
            ColumnName = "ApplicationName", 
            DataType = SqlDbType.NVarChar, 
            DataLength = 50,
            AllowNull = false,
            PropertyName = "ApplicationName"
        },
        new SqlColumn 
        { 
            ColumnName = "EnvironmentName", 
            DataType = SqlDbType.NVarChar, 
            DataLength = 50,
            AllowNull = false,
            PropertyName = "EnvironmentName"
        },
        new SqlColumn 
        { 
            ColumnName = "CorrelationId", 
            DataType = SqlDbType.UniqueIdentifier,
            AllowNull = true,
            PropertyName = "CorrelationId"
        },
        new SqlColumn 
        { 
            ColumnName = "IpAddress", 
            DataType = SqlDbType.NVarChar, 
            DataLength = 50,
            AllowNull = true,
            PropertyName = "IpAddress"
        }
    };
    
    return columnOptions;
}
```

### Enriching Logs with Context

Add contextual information to your logs using enrichers:

```csharp
using Serilog;

var logger = new LoggerConfiguration()
    .Enrich.FromLogContext()
    .Enrich.WithProperty("ApplicationName", "CRM System")
    .Enrich.WithProperty("EnvironmentName", builder.Environment.EnvironmentName)
    .Enrich.WithMachineName()
    .Enrich.WithThreadId()
    .WriteTo.Console()
    .WriteTo.MSSqlServer(/* configuration */)
    .CreateLogger();
```

### Request Logging Middleware

For ASP.NET Core applications, enable HTTP request logging:

```csharp
// In Program.cs, after building the app
app.UseSerilogRequestLogging(options =>
{
    options.MessageTemplate = "HTTP {RequestMethod} {RequestPath} responded {StatusCode} in {Elapsed:0.0000} ms";
    options.EnrichDiagnosticContext = (diagnosticContext, httpContext) =>
    {
        diagnosticContext.Set("RequestHost", httpContext.Request.Host.Value);
        diagnosticContext.Set("RequestScheme", httpContext.Request.Scheme);
        diagnosticContext.Set("UserAgent", httpContext.Request.Headers["User-Agent"].ToString());
        diagnosticContext.Set("ClientIP", httpContext.Connection.RemoteIpAddress?.ToString());
    };
});
```

### Conditional Logging to Different Sinks

You can configure different minimum levels for different sinks:

```csharp
var logger = new LoggerConfiguration()
    .MinimumLevel.Debug()
    .WriteTo.Console(
        restrictedToMinimumLevel: Serilog.Events.LogEventLevel.Information)
    .WriteTo.MSSqlServer(
        connectionString: connectionString,
        sinkOptions: new MSSqlServerSinkOptions { TableName = "Logs" },
        restrictedToMinimumLevel: Serilog.Events.LogEventLevel.Warning)
    .CreateLogger();
```

### Filtering Logs

Filter out specific logs using expressions:

```csharp
using Serilog.Filters;

var logger = new LoggerConfiguration()
    .Filter.ByExcluding(Matching.FromSource("Microsoft.AspNetCore.Hosting"))
    .Filter.ByExcluding(logEvent => 
        logEvent.MessageTemplate.Text.Contains("health check"))
    .WriteTo.Console()
    .WriteTo.MSSqlServer(/* configuration */)
    .CreateLogger();
```

## Best Practices

### 1. Log Levels Appropriately
- **Verbose/Debug**: Detailed diagnostic information, typically disabled in production
- **Information**: General application flow (startup, shutdown, major operations)
- **Warning**: Unexpected situations that don't prevent normal operation
- **Error**: Errors and exceptions that prevent specific operations
- **Fatal**: Critical errors that cause application shutdown

Example:
```csharp
_logger.LogDebug("Cache hit for key: {CacheKey}", cacheKey);
_logger.LogInformation("User {UserId} logged in successfully", userId);
_logger.LogWarning("Request rate limit approaching for user {UserId}", userId);
_logger.LogError(ex, "Failed to process order {OrderId}", orderId);
_logger.LogCritical(ex, "Database connection lost");
```

### 2. Use Structured Properties
Always use structured properties instead of string interpolation:

```csharp
// ❌ Bad - loses structure and incurs unnecessary string allocation
_logger.LogInformation($"User {userId} created order {orderId}");

// ✅ Good - maintains structure and better performance
_logger.LogInformation("User {UserId} created order {OrderId}", userId, orderId);
```

**Note**: String interpolation also has performance implications since the string is always constructed regardless of whether the log level is enabled.

### 3. Include Relevant Context
Add context that helps with debugging:

```csharp
using (LogContext.PushProperty("CorrelationId", correlationId))
using (LogContext.PushProperty("UserId", userId))
{
    _logger.LogInformation("Processing payment for order {OrderId}", orderId);
    // All logs in this scope will include CorrelationId and UserId
}
```

### 4. Performance Considerations
- Avoid logging in tight loops
- Use appropriate log levels to reduce volume in production
- Consider async logging for high-throughput scenarios
- Set appropriate minimum levels for different environments

### 5. Security Best Practices
- Never log sensitive information (passwords, credit cards, API keys)
- Sanitize user input before logging
- Be cautious with personally identifiable information (PII)

```csharp
// ❌ Bad - logs sensitive data (DO NOT DO THIS)
_logger.LogInformation("User login attempt: {Email}, Password: ***", "user@example.com");

// ✅ Good - no sensitive data
_logger.LogInformation("User login attempt for: {Email}", email);
```

### 6. Scalability Guidelines
For high-scale applications:
- **Console**: Use in development and for initial troubleshooting
- **Database**: For structured queries and medium-term storage (7-30 days)
- **Consider Additional Sinks**: 
  - File sinks with rolling for long-term archival
  - Application Insights or similar for production monitoring
  - Elasticsearch or similar for large-scale log aggregation

### 7. Environment-Specific Configuration
Configure different logging behavior per environment:

```csharp
if (builder.Environment.IsDevelopment())
{
    logger = new LoggerConfiguration()
        .MinimumLevel.Debug()
        .WriteTo.Console()
        .CreateLogger();
}
else
{
    logger = new LoggerConfiguration()
        .MinimumLevel.Information()
        .WriteTo.Console()
        .WriteTo.MSSqlServer(/* production configuration */)
        .CreateLogger();
}
```

### 8. Database Maintenance
Regularly maintain your logs table:

```sql
-- Archive old logs (specify columns for better performance)
INSERT INTO LogsArchive (Id, Message, Level, TimeStamp, Exception)
SELECT Id, Message, Level, TimeStamp, Exception
FROM Logs 
WHERE TimeStamp < DATEADD(DAY, -30, GETUTCDATE());

-- Delete archived logs
DELETE FROM Logs 
WHERE TimeStamp < DATEADD(DAY, -30, GETUTCDATE());

-- Create indexes for better query performance
-- Note: Create indexes during low-traffic periods to avoid blocking operations
CREATE INDEX IX_Logs_TimeStamp ON Logs(TimeStamp DESC);
CREATE INDEX IX_Logs_Level ON Logs(Level);
```

### 9. Testing with Logs
Consider using test sinks or in-memory logging during unit tests:

```csharp
// In test setup
var testLogger = new LoggerConfiguration()
    .WriteTo.Debug()
    .CreateLogger();
```

## Conclusion

Structured logging with Serilog provides a powerful foundation for observability in .NET 6 applications. By implementing both console and Microsoft SQL Server sinks, you gain:

1. **Immediate Feedback**: Console logs for real-time monitoring during development and deployment
2. **Persistent Storage**: SQL Server logs for historical analysis and troubleshooting
3. **Query Capabilities**: SQL-based filtering and analysis of log data
4. **Scalability**: Ability to add additional sinks (Application Insights, Elasticsearch, etc.) as your needs grow
5. **Debugging Efficiency**: Rich structured data makes it easier to trace issues across distributed systems
6. **Operational Insights**: Better understanding of application behavior and performance

The combination of structured logging practices with Serilog's flexible sink architecture creates a robust logging infrastructure that supports both development productivity and production reliability. As your application evolves, you can easily extend your logging strategy by adding new sinks, enrichers, and filters without changing your application code.

Remember to:
- Use appropriate log levels
- Include relevant structured properties
- Maintain your log database regularly
- Follow security best practices
- Adjust configurations per environment

With these practices in place, your logging infrastructure will provide valuable insights into your application's behavior, enabling faster debugging, better monitoring, and improved overall system reliability.
