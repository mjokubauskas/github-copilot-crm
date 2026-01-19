# Minimal API with CQRS Pattern in .NET 6 - Complete Implementation Guide

## Table of Contents
1. [Overview](#overview)
2. [Project Setup](#project-setup)
3. [Creating Commands and Queries](#creating-commands-and-queries)
4. [Program.cs Configuration](#programcs-configuration)
5. [Database Connection](#database-connection)
6. [Authentication and Authorization](#authentication-and-authorization)
7. [Optional Features](#optional-features)
8. [Conclusion](#conclusion)

---

## Overview

### What are Minimal APIs?

Minimal APIs in .NET 6 provide a streamlined approach to building HTTP APIs with minimal ceremony and configuration. They are designed to:

- Reduce boilerplate code and startup complexity
- Provide a lightweight alternative to traditional ASP.NET Core MVC/Web API
- Enable rapid prototyping and microservices development
- Offer top-tier performance with minimal overhead
- Support modern C# features like top-level statements and records

**Key Benefits:**
- **Simplicity**: Less code, fewer abstractions
- **Performance**: Faster startup times and reduced memory footprint
- **Modern**: Leverages latest C# language features
- **Scalable**: Suitable for both small APIs and large-scale microservices

### What is CQRS?

CQRS (Command Query Responsibility Segregation) is a pattern that separates read operations (Queries) from write operations (Commands):

- **Commands**: Modify state (Create, Update, Delete) and return no data or only success/failure
- **Queries**: Retrieve data without side effects and return DTOs

**Key Benefits:**
- **Scalability**: Scale reads and writes independently
- **Performance**: Optimize queries without affecting commands
- **Clarity**: Clear separation of concerns
- **Flexibility**: Different models for reading and writing
- **Maintainability**: Easier to understand and modify

### How They Work Together

Combining Minimal APIs with CQRS creates a powerful, lightweight architecture:

1. **Minimal API endpoints** serve as the entry points for HTTP requests
2. **MediatR** acts as the mediator, routing requests to appropriate handlers
3. **Commands/Queries** encapsulate the request data and intent
4. **Handlers** contain the business logic for processing operations
5. **Dapper** provides fast, lightweight data access

This architecture is ideal for:
- Microservices
- Cloud-native applications
- High-performance APIs
- Event-driven systems

---

## Project Setup

### Step 1: Create a New Minimal API Project

Use the .NET CLI to create a new Web API project:

```bash
# Create a new Minimal API project
dotnet new web -n ProductCatalogApi
cd ProductCatalogApi

# Create solution file (optional)
dotnet new sln -n ProductCatalog
dotnet sln add ProductCatalogApi/ProductCatalogApi.csproj
```

### Step 2: Install Required NuGet Packages

Install the essential packages for implementing CQRS with Minimal API:

```bash
# MediatR for CQRS pattern
dotnet add package MediatR
dotnet add package MediatR.Extensions.Microsoft.DependencyInjection

# Dapper for data access
dotnet add package Dapper
dotnet add package Microsoft.Data.SqlClient

# Authentication
dotnet add package Microsoft.AspNetCore.Authentication.JwtBearer
dotnet add package Microsoft.Identity.Web

# Optional but recommended
dotnet add package FluentValidation
dotnet add package FluentValidation.DependencyInjectionExtensions
dotnet add package Serilog.AspNetCore
```

### Step 3: Project Structure

Organize your project with a clean structure:

```
ProductCatalogApi/
├── Commands/
│   ├── CreateProduct/
│   │   ├── CreateProductCommand.cs
│   │   └── CreateProductCommandHandler.cs
│   ├── UpdateProduct/
│   │   ├── UpdateProductCommand.cs
│   │   └── UpdateProductCommandHandler.cs
│   └── DeleteProduct/
│       ├── DeleteProductCommand.cs
│       └── DeleteProductCommandHandler.cs
├── Queries/
│   ├── GetProducts/
│   │   ├── GetProductsQuery.cs
│   │   └── GetProductsQueryHandler.cs
│   └── GetProductById/
│       ├── GetProductByIdQuery.cs
│       └── GetProductByIdQueryHandler.cs
├── Models/
│   ├── Product.cs
│   └── ProductDto.cs
├── Data/
│   ├── IDbConnectionFactory.cs
│   └── SqlConnectionFactory.cs
├── Middleware/
│   └── ErrorHandlingMiddleware.cs
└── Program.cs
```

---

## Creating Commands and Queries

### Commands for Write Operations

Commands represent actions that modify the application state. They should be named as verbs describing the action.

#### Example: Create Product Command

**CreateProductCommand.cs**
```csharp
using MediatR;

namespace ProductCatalogApi.Commands.CreateProduct;

public record CreateProductCommand(
    string Name,
    string Description,
    decimal Price,
    int StockQuantity,
    string Category
) : IRequest<CreateProductResult>;

public record CreateProductResult(
    int ProductId,
    bool Success,
    string Message
);
```

**CreateProductCommandHandler.cs**
```csharp
using Dapper;
using MediatR;
using ProductCatalogApi.Data;

namespace ProductCatalogApi.Commands.CreateProduct;

public class CreateProductCommandHandler : IRequestHandler<CreateProductCommand, CreateProductResult>
{
    private readonly IDbConnectionFactory _connectionFactory;
    private readonly ILogger<CreateProductCommandHandler> _logger;

    public CreateProductCommandHandler(
        IDbConnectionFactory connectionFactory,
        ILogger<CreateProductCommandHandler> logger)
    {
        _connectionFactory = connectionFactory;
        _logger = logger;
    }

    public async Task<CreateProductResult> Handle(
        CreateProductCommand request,
        CancellationToken cancellationToken)
    {
        try
        {
            using var connection = _connectionFactory.CreateConnection();
            
            const string sql = @"
                INSERT INTO Products (Name, Description, Price, StockQuantity, Category, CreatedAt, IsActive)
                VALUES (@Name, @Description, @Price, @StockQuantity, @Category, GETUTCDATE(), 1);
                SELECT CAST(SCOPE_IDENTITY() AS INT);";

            var productId = await connection.ExecuteScalarAsync<int>(sql, new
            {
                request.Name,
                request.Description,
                request.Price,
                request.StockQuantity,
                request.Category
            });

            _logger.LogInformation(
                "Product created successfully with ID: {ProductId}", 
                productId);

            return new CreateProductResult(
                productId,
                true,
                "Product created successfully");
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error creating product: {Name}", request.Name);
            return new CreateProductResult(
                0,
                false,
                $"Failed to create product: {ex.Message}");
        }
    }
}
```

#### Example: Update Product Command

**UpdateProductCommand.cs**
```csharp
using MediatR;

namespace ProductCatalogApi.Commands.UpdateProduct;

public record UpdateProductCommand(
    int ProductId,
    string Name,
    string Description,
    decimal Price,
    int StockQuantity,
    string Category,
    bool IsActive
) : IRequest<UpdateProductResult>;

public record UpdateProductResult(
    bool Success,
    string Message
);
```

**UpdateProductCommandHandler.cs**
```csharp
using Dapper;
using MediatR;
using ProductCatalogApi.Data;

namespace ProductCatalogApi.Commands.UpdateProduct;

public class UpdateProductCommandHandler : IRequestHandler<UpdateProductCommand, UpdateProductResult>
{
    private readonly IDbConnectionFactory _connectionFactory;
    private readonly ILogger<UpdateProductCommandHandler> _logger;

    public UpdateProductCommandHandler(
        IDbConnectionFactory connectionFactory,
        ILogger<UpdateProductCommandHandler> logger)
    {
        _connectionFactory = connectionFactory;
        _logger = logger;
    }

    public async Task<UpdateProductResult> Handle(
        UpdateProductCommand request,
        CancellationToken cancellationToken)
    {
        try
        {
            using var connection = _connectionFactory.CreateConnection();
            
            const string sql = @"
                UPDATE Products 
                SET Name = @Name,
                    Description = @Description,
                    Price = @Price,
                    StockQuantity = @StockQuantity,
                    Category = @Category,
                    IsActive = @IsActive,
                    UpdatedAt = GETUTCDATE()
                WHERE ProductId = @ProductId";

            var rowsAffected = await connection.ExecuteAsync(sql, request);

            if (rowsAffected == 0)
            {
                return new UpdateProductResult(
                    false,
                    $"Product with ID {request.ProductId} not found");
            }

            _logger.LogInformation(
                "Product updated successfully: {ProductId}", 
                request.ProductId);

            return new UpdateProductResult(
                true,
                "Product updated successfully");
        }
        catch (Exception ex)
        {
            _logger.LogError(
                ex, 
                "Error updating product: {ProductId}", 
                request.ProductId);
            return new UpdateProductResult(
                false,
                $"Failed to update product: {ex.Message}");
        }
    }
}
```

#### Example: Delete Product Command

**DeleteProductCommand.cs**
```csharp
using MediatR;

namespace ProductCatalogApi.Commands.DeleteProduct;

public record DeleteProductCommand(int ProductId) : IRequest<DeleteProductResult>;

public record DeleteProductResult(
    bool Success,
    string Message
);
```

**DeleteProductCommandHandler.cs**
```csharp
using Dapper;
using MediatR;
using ProductCatalogApi.Data;

namespace ProductCatalogApi.Commands.DeleteProduct;

public class DeleteProductCommandHandler : IRequestHandler<DeleteProductCommand, DeleteProductResult>
{
    private readonly IDbConnectionFactory _connectionFactory;
    private readonly ILogger<DeleteProductCommandHandler> _logger;

    public DeleteProductCommandHandler(
        IDbConnectionFactory connectionFactory,
        ILogger<DeleteProductCommandHandler> logger)
    {
        _connectionFactory = connectionFactory;
        _logger = logger;
    }

    public async Task<DeleteProductResult> Handle(
        DeleteProductCommand request,
        CancellationToken cancellationToken)
    {
        try
        {
            using var connection = _connectionFactory.CreateConnection();
            
            // Soft delete - mark as inactive
            const string sql = @"
                UPDATE Products 
                SET IsActive = 0, 
                    UpdatedAt = GETUTCDATE()
                WHERE ProductId = @ProductId";

            var rowsAffected = await connection.ExecuteAsync(
                sql, 
                new { request.ProductId });

            if (rowsAffected == 0)
            {
                return new DeleteProductResult(
                    false,
                    $"Product with ID {request.ProductId} not found");
            }

            _logger.LogInformation(
                "Product deleted successfully: {ProductId}", 
                request.ProductId);

            return new DeleteProductResult(
                true,
                "Product deleted successfully");
        }
        catch (Exception ex)
        {
            _logger.LogError(
                ex, 
                "Error deleting product: {ProductId}", 
                request.ProductId);
            return new DeleteProductResult(
                false,
                $"Failed to delete product: {ex.Message}");
        }
    }
}
```

### Queries for Read Operations

Queries retrieve data without side effects. They should be named as questions describing what data is being retrieved.

#### Example: Get Products Query

**GetProductsQuery.cs**
```csharp
using MediatR;
using ProductCatalogApi.Models;

namespace ProductCatalogApi.Queries.GetProducts;

public record GetProductsQuery(
    string? SearchTerm,
    string? Category,
    decimal? MinPrice,
    decimal? MaxPrice,
    string? SortBy,
    bool SortDescending,
    int PageNumber,
    int PageSize
) : IRequest<GetProductsResult>;

public record GetProductsResult(
    IEnumerable<ProductDto> Products,
    int TotalCount,
    int PageNumber,
    int PageSize
);
```

**GetProductsQueryHandler.cs**
```csharp
using Dapper;
using MediatR;
using ProductCatalogApi.Data;
using ProductCatalogApi.Models;
using System.Text;

namespace ProductCatalogApi.Queries.GetProducts;

public class GetProductsQueryHandler : IRequestHandler<GetProductsQuery, GetProductsResult>
{
    private readonly IDbConnectionFactory _connectionFactory;
    private readonly ILogger<GetProductsQueryHandler> _logger;

    public GetProductsQueryHandler(
        IDbConnectionFactory connectionFactory,
        ILogger<GetProductsQueryHandler> logger)
    {
        _connectionFactory = connectionFactory;
        _logger = logger;
    }

    public async Task<GetProductsResult> Handle(
        GetProductsQuery request,
        CancellationToken cancellationToken)
    {
        try
        {
            using var connection = _connectionFactory.CreateConnection();
            
            var whereConditions = new List<string> { "IsActive = 1" };
            var parameters = new DynamicParameters();

            // Search filter
            if (!string.IsNullOrWhiteSpace(request.SearchTerm))
            {
                whereConditions.Add("(Name LIKE @SearchTerm OR Description LIKE @SearchTerm)");
                parameters.Add("SearchTerm", $"%{request.SearchTerm}%");
            }

            // Category filter
            if (!string.IsNullOrWhiteSpace(request.Category))
            {
                whereConditions.Add("Category = @Category");
                parameters.Add("Category", request.Category);
            }

            // Price range filter
            if (request.MinPrice.HasValue)
            {
                whereConditions.Add("Price >= @MinPrice");
                parameters.Add("MinPrice", request.MinPrice.Value);
            }

            if (request.MaxPrice.HasValue)
            {
                whereConditions.Add("Price <= @MaxPrice");
                parameters.Add("MaxPrice", request.MaxPrice.Value);
            }

            var whereClause = string.Join(" AND ", whereConditions);

            // Get total count
            var countSql = $"SELECT COUNT(*) FROM Products WHERE {whereClause}";
            var totalCount = await connection.ExecuteScalarAsync<int>(countSql, parameters);

            // Build sorted query with pagination
            var sortColumn = request.SortBy?.ToLower() switch
            {
                "name" => "Name",
                "price" => "Price",
                "category" => "Category",
                "stock" => "StockQuantity",
                _ => "CreatedAt"
            };

            var sortDirection = request.SortDescending ? "DESC" : "ASC";
            var offset = (request.PageNumber - 1) * request.PageSize;

            var sql = $@"
                SELECT ProductId, Name, Description, Price, StockQuantity, Category, IsActive, CreatedAt, UpdatedAt
                FROM Products
                WHERE {whereClause}
                ORDER BY {sortColumn} {sortDirection}
                OFFSET @Offset ROWS
                FETCH NEXT @PageSize ROWS ONLY";

            parameters.Add("Offset", offset);
            parameters.Add("PageSize", request.PageSize);

            var products = await connection.QueryAsync<ProductDto>(sql, parameters);

            _logger.LogInformation(
                "Retrieved {Count} products (Page {Page} of {Total})",
                products.Count(),
                request.PageNumber,
                (totalCount + request.PageSize - 1) / request.PageSize);

            return new GetProductsResult(
                products,
                totalCount,
                request.PageNumber,
                request.PageSize);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error retrieving products");
            throw;
        }
    }
}
```

#### Example: Get Product By ID Query

**GetProductByIdQuery.cs**
```csharp
using MediatR;
using ProductCatalogApi.Models;

namespace ProductCatalogApi.Queries.GetProductById;

public record GetProductByIdQuery(int ProductId) : IRequest<ProductDto?>;
```

**GetProductByIdQueryHandler.cs**
```csharp
using Dapper;
using MediatR;
using ProductCatalogApi.Data;
using ProductCatalogApi.Models;

namespace ProductCatalogApi.Queries.GetProductById;

public class GetProductByIdQueryHandler : IRequestHandler<GetProductByIdQuery, ProductDto?>
{
    private readonly IDbConnectionFactory _connectionFactory;
    private readonly ILogger<GetProductByIdQueryHandler> _logger;

    public GetProductByIdQueryHandler(
        IDbConnectionFactory connectionFactory,
        ILogger<GetProductByIdQueryHandler> logger)
    {
        _connectionFactory = connectionFactory;
        _logger = logger;
    }

    public async Task<ProductDto?> Handle(
        GetProductByIdQuery request,
        CancellationToken cancellationToken)
    {
        try
        {
            using var connection = _connectionFactory.CreateConnection();
            
            const string sql = @"
                SELECT ProductId, Name, Description, Price, StockQuantity, Category, IsActive, CreatedAt, UpdatedAt
                FROM Products
                WHERE ProductId = @ProductId AND IsActive = 1";

            var product = await connection.QuerySingleOrDefaultAsync<ProductDto>(
                sql,
                new { request.ProductId });

            if (product != null)
            {
                _logger.LogInformation(
                    "Retrieved product: {ProductId}", 
                    request.ProductId);
            }
            else
            {
                _logger.LogWarning(
                    "Product not found: {ProductId}", 
                    request.ProductId);
            }

            return product;
        }
        catch (Exception ex)
        {
            _logger.LogError(
                ex, 
                "Error retrieving product: {ProductId}", 
                request.ProductId);
            throw;
        }
    }
}
```

### Models

**Product.cs** (Domain Model)
```csharp
namespace ProductCatalogApi.Models;

public class Product
{
    public int ProductId { get; set; }
    public string Name { get; set; } = string.Empty;
    public string Description { get; set; } = string.Empty;
    public decimal Price { get; set; }
    public int StockQuantity { get; set; }
    public string Category { get; set; } = string.Empty;
    public bool IsActive { get; set; } = true;
    public DateTime CreatedAt { get; set; }
    public DateTime? UpdatedAt { get; set; }
}
```

**ProductDto.cs** (Data Transfer Object)
```csharp
namespace ProductCatalogApi.Models;

public record ProductDto(
    int ProductId,
    string Name,
    string Description,
    decimal Price,
    int StockQuantity,
    string Category,
    bool IsActive,
    DateTime CreatedAt,
    DateTime? UpdatedAt
);
```

---

## Program.cs Configuration

The `Program.cs` file is where we configure services, middleware, and map our API endpoints.

**Complete Program.cs Example**

```csharp
using MediatR;
using Microsoft.AspNetCore.Authentication.JwtBearer;
using Microsoft.Identity.Web;
using ProductCatalogApi.Commands.CreateProduct;
using ProductCatalogApi.Commands.UpdateProduct;
using ProductCatalogApi.Commands.DeleteProduct;
using ProductCatalogApi.Queries.GetProducts;
using ProductCatalogApi.Queries.GetProductById;
using ProductCatalogApi.Data;
using ProductCatalogApi.Middleware;
using Serilog;

var builder = WebApplication.CreateBuilder(args);

// Configure Serilog
Log.Logger = new LoggerConfiguration()
    .ReadFrom.Configuration(builder.Configuration)
    .Enrich.FromLogContext()
    .WriteTo.Console()
    .CreateLogger();

builder.Host.UseSerilog();

// Add services to the container
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

// Register MediatR
builder.Services.AddMediatR(cfg => 
    cfg.RegisterServicesFromAssembly(typeof(Program).Assembly));

// Register database connection factory
builder.Services.AddSingleton<IDbConnectionFactory>(sp =>
    new SqlConnectionFactory(builder.Configuration.GetConnectionString("DefaultConnection")!));

// Configure Authentication
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.Authority = builder.Configuration["Authentication:Authority"];
        options.Audience = builder.Configuration["Authentication:Audience"];
        options.TokenValidationParameters = new()
        {
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidateLifetime = true,
            ValidateIssuerSigningKey = true
        };
    });

// Or use Microsoft Entra ID (Azure AD)
// builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
//     .AddMicrosoftIdentityWebApi(builder.Configuration.GetSection("AzureAd"));

builder.Services.AddAuthorization();

// Add CORS if needed
builder.Services.AddCors(options =>
{
    options.AddPolicy("AllowAll", policy =>
    {
        policy.AllowAnyOrigin()
              .AllowAnyMethod()
              .AllowAnyHeader();
    });
});

var app = builder.Build();

// Configure middleware pipeline
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();

// Add custom error handling middleware
app.UseMiddleware<ErrorHandlingMiddleware>();

app.UseCors("AllowAll");

app.UseAuthentication();
app.UseAuthorization();

// Map API Endpoints

// Health check
app.MapGet("/health", () => Results.Ok(new { status = "healthy", timestamp = DateTime.UtcNow }))
    .WithName("HealthCheck")
    .WithTags("Health");

// Product Endpoints

// GET: Get all products with filtering, sorting, and pagination
app.MapGet("/api/products", async (
    IMediator mediator,
    string? searchTerm,
    string? category,
    decimal? minPrice,
    decimal? maxPrice,
    string? sortBy,
    bool sortDescending = false,
    int pageNumber = 1,
    int pageSize = 10) =>
{
    var query = new GetProductsQuery(
        searchTerm,
        category,
        minPrice,
        maxPrice,
        sortBy,
        sortDescending,
        pageNumber,
        pageSize);

    var result = await mediator.Send(query);
    
    return Results.Ok(new
    {
        data = result.Products,
        pagination = new
        {
            result.TotalCount,
            result.PageNumber,
            result.PageSize,
            TotalPages = (result.TotalCount + result.PageSize - 1) / result.PageSize
        }
    });
})
.WithName("GetProducts")
.WithTags("Products")
.Produces<object>(StatusCodes.Status200OK);

// GET: Get product by ID
app.MapGet("/api/products/{id:int}", async (
    int id,
    IMediator mediator) =>
{
    var query = new GetProductByIdQuery(id);
    var product = await mediator.Send(query);

    return product is not null
        ? Results.Ok(product)
        : Results.NotFound(new { message = $"Product with ID {id} not found" });
})
.WithName("GetProductById")
.WithTags("Products")
.Produces<object>(StatusCodes.Status200OK)
.Produces(StatusCodes.Status404NotFound);

// POST: Create a new product (requires authentication)
app.MapPost("/api/products", async (
    CreateProductCommand command,
    IMediator mediator) =>
{
    var result = await mediator.Send(command);

    return result.Success
        ? Results.Created($"/api/products/{result.ProductId}", result)
        : Results.BadRequest(result);
})
.WithName("CreateProduct")
.WithTags("Products")
.RequireAuthorization()
.Produces<CreateProductResult>(StatusCodes.Status201Created)
.Produces(StatusCodes.Status400BadRequest)
.Produces(StatusCodes.Status401Unauthorized);

// PUT: Update an existing product (requires authentication)
app.MapPut("/api/products/{id:int}", async (
    int id,
    UpdateProductCommand command,
    IMediator mediator) =>
{
    if (id != command.ProductId)
    {
        return Results.BadRequest(new { message = "Product ID mismatch" });
    }

    var result = await mediator.Send(command);

    return result.Success
        ? Results.Ok(result)
        : Results.NotFound(result);
})
.WithName("UpdateProduct")
.WithTags("Products")
.RequireAuthorization()
.Produces<UpdateProductResult>(StatusCodes.Status200OK)
.Produces(StatusCodes.Status400BadRequest)
.Produces(StatusCodes.Status401Unauthorized)
.Produces(StatusCodes.Status404NotFound);

// DELETE: Delete a product (requires authentication)
app.MapDelete("/api/products/{id:int}", async (
    int id,
    IMediator mediator) =>
{
    var command = new DeleteProductCommand(id);
    var result = await mediator.Send(command);

    return result.Success
        ? Results.Ok(result)
        : Results.NotFound(result);
})
.WithName("DeleteProduct")
.WithTags("Products")
.RequireAuthorization()
.Produces<DeleteProductResult>(StatusCodes.Status200OK)
.Produces(StatusCodes.Status401Unauthorized)
.Produces(StatusCodes.Status404NotFound);

// Run the application
try
{
    Log.Information("Starting Product Catalog API");
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

---

## Database Connection

### SQL Schema

Create the database schema for the Products table:

**Create Database and Table**

```sql
-- Create Database
CREATE DATABASE ProductCatalogDb;
GO

USE ProductCatalogDb;
GO

-- Create Products Table
CREATE TABLE Products (
    ProductId INT IDENTITY(1,1) PRIMARY KEY,
    Name NVARCHAR(200) NOT NULL,
    Description NVARCHAR(1000) NOT NULL,
    Price DECIMAL(18, 2) NOT NULL CHECK (Price >= 0),
    StockQuantity INT NOT NULL DEFAULT 0 CHECK (StockQuantity >= 0),
    Category NVARCHAR(100) NOT NULL,
    IsActive BIT NOT NULL DEFAULT 1,
    CreatedAt DATETIME2 NOT NULL DEFAULT GETUTCDATE(),
    UpdatedAt DATETIME2 NULL,
    CONSTRAINT CK_Products_Price CHECK (Price >= 0),
    CONSTRAINT CK_Products_Stock CHECK (StockQuantity >= 0)
);
GO

-- Create indexes for better query performance
CREATE INDEX IX_Products_Category ON Products(Category) WHERE IsActive = 1;
CREATE INDEX IX_Products_Price ON Products(Price) WHERE IsActive = 1;
CREATE INDEX IX_Products_Name ON Products(Name) WHERE IsActive = 1;
CREATE INDEX IX_Products_CreatedAt ON Products(CreatedAt DESC) WHERE IsActive = 1;
GO

-- Insert sample data
INSERT INTO Products (Name, Description, Price, StockQuantity, Category, CreatedAt)
VALUES 
    ('Laptop Pro 15"', 'High-performance laptop with 16GB RAM and 512GB SSD', 1299.99, 50, 'Electronics', GETUTCDATE()),
    ('Wireless Mouse', 'Ergonomic wireless mouse with USB receiver', 29.99, 200, 'Electronics', GETUTCDATE()),
    ('Office Chair', 'Comfortable ergonomic office chair with lumbar support', 249.99, 75, 'Furniture', GETUTCDATE()),
    ('Standing Desk', 'Adjustable height standing desk', 499.99, 30, 'Furniture', GETUTCDATE()),
    ('Mechanical Keyboard', 'RGB backlit mechanical keyboard with Cherry MX switches', 149.99, 100, 'Electronics', GETUTCDATE()),
    ('Monitor 27"', '4K Ultra HD monitor with HDR support', 399.99, 60, 'Electronics', GETUTCDATE()),
    ('Desk Lamp', 'LED desk lamp with adjustable brightness', 39.99, 150, 'Furniture', GETUTCDATE()),
    ('USB-C Hub', '7-in-1 USB-C hub with HDMI and card readers', 49.99, 120, 'Electronics', GETUTCDATE());
GO
```

### Dapper Integration

**IDbConnectionFactory.cs**

```csharp
using System.Data;

namespace ProductCatalogApi.Data;

public interface IDbConnectionFactory
{
    IDbConnection CreateConnection();
}
```

**SqlConnectionFactory.cs**

```csharp
using Microsoft.Data.SqlClient;
using System.Data;

namespace ProductCatalogApi.Data;

public class SqlConnectionFactory : IDbConnectionFactory
{
    private readonly string _connectionString;

    public SqlConnectionFactory(string connectionString)
    {
        _connectionString = connectionString ?? throw new ArgumentNullException(nameof(connectionString));
    }

    public IDbConnection CreateConnection()
    {
        return new SqlConnection(_connectionString);
    }
}
```

**Connection String Configuration (appsettings.json)**

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database=ProductCatalogDb;Trusted_Connection=True;TrustServerCertificate=True;"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*"
}
```

### CRUD Operations with Dapper

The CRUD operations are already implemented in the command and query handlers above. Here's a summary:

**Create (INSERT)**
```csharp
const string sql = @"
    INSERT INTO Products (Name, Description, Price, StockQuantity, Category, CreatedAt, IsActive)
    VALUES (@Name, @Description, @Price, @StockQuantity, @Category, GETUTCDATE(), 1);
    SELECT CAST(SCOPE_IDENTITY() AS INT);";

var productId = await connection.ExecuteScalarAsync<int>(sql, command);
```

**Read (SELECT)**
```csharp
const string sql = @"
    SELECT ProductId, Name, Description, Price, StockQuantity, Category, IsActive, CreatedAt, UpdatedAt
    FROM Products
    WHERE ProductId = @ProductId AND IsActive = 1";

var product = await connection.QuerySingleOrDefaultAsync<ProductDto>(sql, new { ProductId = id });
```

**Update (UPDATE)**
```csharp
const string sql = @"
    UPDATE Products 
    SET Name = @Name,
        Description = @Description,
        Price = @Price,
        StockQuantity = @StockQuantity,
        Category = @Category,
        IsActive = @IsActive,
        UpdatedAt = GETUTCDATE()
    WHERE ProductId = @ProductId";

var rowsAffected = await connection.ExecuteAsync(sql, command);
```

**Delete (Soft Delete - UPDATE)**
```csharp
const string sql = @"
    UPDATE Products 
    SET IsActive = 0, 
        UpdatedAt = GETUTCDATE()
    WHERE ProductId = @ProductId";

var rowsAffected = await connection.ExecuteAsync(sql, new { ProductId = id });
```

---

## Authentication and Authorization

### Option 1: JWT Token Authentication

**appsettings.json Configuration**

```json
{
  "Authentication": {
    "Authority": "https://your-identity-server.com",
    "Audience": "product-catalog-api",
    "Issuer": "https://your-identity-server.com"
  },
  "Jwt": {
    "Key": "YourSuperSecretKeyThatIsAtLeast32CharactersLong!",
    "Issuer": "ProductCatalogApi",
    "Audience": "ProductCatalogApiUsers",
    "ExpiryMinutes": 60
  }
}
```

**JWT Configuration in Program.cs**

```csharp
using Microsoft.AspNetCore.Authentication.JwtBearer;
using Microsoft.IdentityModel.Tokens;
using System.Text;

// Configure JWT Authentication
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        var jwtSettings = builder.Configuration.GetSection("Jwt");
        var key = Encoding.UTF8.GetBytes(jwtSettings["Key"]!);

        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidateLifetime = true,
            ValidateIssuerSigningKey = true,
            ValidIssuer = jwtSettings["Issuer"],
            ValidAudience = jwtSettings["Audience"],
            IssuerSigningKey = new SymmetricSecurityKey(key),
            ClockSkew = TimeSpan.Zero
        };

        options.Events = new JwtBearerEvents
        {
            OnAuthenticationFailed = context =>
            {
                if (context.Exception.GetType() == typeof(SecurityTokenExpiredException))
                {
                    context.Response.Headers.Add("Token-Expired", "true");
                }
                return Task.CompletedTask;
            }
        };
    });

builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("AdminOnly", policy => 
        policy.RequireClaim("role", "Admin"));
    
    options.AddPolicy("UserOrAdmin", policy =>
        policy.RequireClaim("role", "User", "Admin"));
});
```

**Example: Generating JWT Tokens**

```csharp
using Microsoft.IdentityModel.Tokens;
using System.IdentityModel.Tokens.Jwt;
using System.Security.Claims;
using System.Text;

public class JwtTokenService
{
    private readonly IConfiguration _configuration;

    public JwtTokenService(IConfiguration configuration)
    {
        _configuration = configuration;
    }

    public string GenerateToken(string userId, string email, string role)
    {
        var jwtSettings = _configuration.GetSection("Jwt");
        var key = new SymmetricSecurityKey(
            Encoding.UTF8.GetBytes(jwtSettings["Key"]!));
        
        var credentials = new SigningCredentials(
            key, 
            SecurityAlgorithms.HmacSha256);

        var claims = new[]
        {
            new Claim(JwtRegisteredClaimNames.Sub, userId),
            new Claim(JwtRegisteredClaimNames.Email, email),
            new Claim(JwtRegisteredClaimNames.Jti, Guid.NewGuid().ToString()),
            new Claim("role", role)
        };

        var token = new JwtSecurityToken(
            issuer: jwtSettings["Issuer"],
            audience: jwtSettings["Audience"],
            claims: claims,
            expires: DateTime.UtcNow.AddMinutes(
                int.Parse(jwtSettings["ExpiryMinutes"]!)),
            signingCredentials: credentials);

        return new JwtSecurityTokenHandler().WriteToken(token);
    }
}
```

**Login Endpoint Example**

```csharp
app.MapPost("/api/auth/login", async (
    LoginRequest request,
    JwtTokenService tokenService) =>
{
    // Validate credentials (implement your own validation logic)
    if (request.Username == "admin" && request.Password == "password")
    {
        var token = tokenService.GenerateToken(
            "1", 
            "admin@example.com", 
            "Admin");
        
        return Results.Ok(new { token, expiresIn = 3600 });
    }

    return Results.Unauthorized();
})
.WithName("Login")
.WithTags("Authentication")
.AllowAnonymous();

public record LoginRequest(string Username, string Password);
```

### Option 2: Microsoft Entra ID (Azure AD) Authentication

**appsettings.json Configuration**

```json
{
  "AzureAd": {
    "Instance": "https://login.microsoftonline.com/",
    "Domain": "yourdomain.onmicrosoft.com",
    "TenantId": "your-tenant-id",
    "ClientId": "your-client-id",
    "Audience": "api://your-client-id",
    "Scopes": "access_as_user"
  }
}
```

**Microsoft Entra ID Configuration in Program.cs**

```csharp
using Microsoft.AspNetCore.Authentication.JwtBearer;
using Microsoft.Identity.Web;

// Configure Microsoft Entra ID Authentication
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddMicrosoftIdentityWebApi(builder.Configuration.GetSection("AzureAd"));

builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("ReadAccess", policy =>
        policy.RequireScope("Products.Read"));
    
    options.AddPolicy("WriteAccess", policy =>
        policy.RequireScope("Products.Write"));
});
```

**Securing Endpoints with Policies**

```csharp
// Require specific scope for endpoint
app.MapGet("/api/products", async (IMediator mediator) =>
{
    // Implementation
})
.RequireAuthorization("ReadAccess");

// Require authentication but no specific policy
app.MapPost("/api/products", async (CreateProductCommand command, IMediator mediator) =>
{
    // Implementation
})
.RequireAuthorization();

// Multiple policies
app.MapDelete("/api/products/{id:int}", async (int id, IMediator mediator) =>
{
    // Implementation
})
.RequireAuthorization("WriteAccess");
```

---

## Optional Features

### 1. Error Handling Middleware

Create a centralized error handling middleware to catch and handle exceptions gracefully.

**ErrorHandlingMiddleware.cs**

```csharp
using System.Net;
using System.Text.Json;

namespace ProductCatalogApi.Middleware;

public class ErrorHandlingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<ErrorHandlingMiddleware> _logger;

    public ErrorHandlingMiddleware(
        RequestDelegate next,
        ILogger<ErrorHandlingMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        try
        {
            await _next(context);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "An unhandled exception occurred");
            await HandleExceptionAsync(context, ex);
        }
    }

    private static Task HandleExceptionAsync(HttpContext context, Exception exception)
    {
        context.Response.ContentType = "application/json";
        context.Response.StatusCode = exception switch
        {
            ArgumentNullException => (int)HttpStatusCode.BadRequest,
            ArgumentException => (int)HttpStatusCode.BadRequest,
            KeyNotFoundException => (int)HttpStatusCode.NotFound,
            UnauthorizedAccessException => (int)HttpStatusCode.Unauthorized,
            _ => (int)HttpStatusCode.InternalServerError
        };

        var response = new ErrorResponse
        {
            StatusCode = context.Response.StatusCode,
            Message = exception.Message,
            Details = exception.StackTrace,
            Timestamp = DateTime.UtcNow
        };

        var options = new JsonSerializerOptions
        {
            PropertyNamingPolicy = JsonNamingPolicy.CamelCase
        };

        return context.Response.WriteAsync(
            JsonSerializer.Serialize(response, options));
    }
}

public class ErrorResponse
{
    public int StatusCode { get; set; }
    public string Message { get; set; } = string.Empty;
    public string? Details { get; set; }
    public DateTime Timestamp { get; set; }
}
```

### 2. Request Validation with FluentValidation

Add validation for incoming requests to ensure data integrity.

**CreateProductCommandValidator.cs**

```csharp
using FluentValidation;
using ProductCatalogApi.Commands.CreateProduct;

namespace ProductCatalogApi.Validators;

public class CreateProductCommandValidator : AbstractValidator<CreateProductCommand>
{
    public CreateProductCommandValidator()
    {
        RuleFor(x => x.Name)
            .NotEmpty().WithMessage("Product name is required")
            .MaximumLength(200).WithMessage("Product name must not exceed 200 characters");

        RuleFor(x => x.Description)
            .NotEmpty().WithMessage("Description is required")
            .MaximumLength(1000).WithMessage("Description must not exceed 1000 characters");

        RuleFor(x => x.Price)
            .GreaterThan(0).WithMessage("Price must be greater than zero");

        RuleFor(x => x.StockQuantity)
            .GreaterThanOrEqualTo(0).WithMessage("Stock quantity cannot be negative");

        RuleFor(x => x.Category)
            .NotEmpty().WithMessage("Category is required")
            .MaximumLength(100).WithMessage("Category must not exceed 100 characters");
    }
}
```

**Register FluentValidation in Program.cs**

```csharp
using FluentValidation;

// Register validators
builder.Services.AddValidatorsFromAssembly(typeof(Program).Assembly);

// Add validation behavior to MediatR pipeline
builder.Services.AddTransient(typeof(IPipelineBehavior<,>), typeof(ValidationBehavior<,>));
```

**ValidationBehavior.cs**

```csharp
using FluentValidation;
using MediatR;

namespace ProductCatalogApi.Behaviors;

public class ValidationBehavior<TRequest, TResponse> : IPipelineBehavior<TRequest, TResponse>
    where TRequest : IRequest<TResponse>
{
    private readonly IEnumerable<IValidator<TRequest>> _validators;

    public ValidationBehavior(IEnumerable<IValidator<TRequest>> validators)
    {
        _validators = validators;
    }

    public async Task<TResponse> Handle(
        TRequest request,
        RequestHandlerDelegate<TResponse> next,
        CancellationToken cancellationToken)
    {
        if (!_validators.Any())
        {
            return await next();
        }

        var context = new ValidationContext<TRequest>(request);

        var validationResults = await Task.WhenAll(
            _validators.Select(v => v.ValidateAsync(context, cancellationToken)));

        var failures = validationResults
            .SelectMany(r => r.Errors)
            .Where(f => f != null)
            .ToList();

        if (failures.Any())
        {
            throw new ValidationException(failures);
        }

        return await next();
    }
}
```

### 3. Sorting, Filtering, and Searching

The `GetProductsQuery` handler already implements comprehensive filtering, searching, and sorting:

**Features Included:**

1. **Search**: Full-text search across Name and Description
   ```csharp
   ?searchTerm=laptop
   ```

2. **Category Filter**: Filter by specific category
   ```csharp
   ?category=Electronics
   ```

3. **Price Range Filter**: Filter by minimum and maximum price
   ```csharp
   ?minPrice=50&maxPrice=500
   ```

4. **Sorting**: Sort by multiple fields with ascending/descending order
   ```csharp
   ?sortBy=price&sortDescending=true
   ```

5. **Pagination**: Navigate through results with page numbers and sizes
   ```csharp
   ?pageNumber=1&pageSize=20
   ```

**Complete Example Request:**

```
GET /api/products?searchTerm=laptop&category=Electronics&minPrice=1000&maxPrice=2000&sortBy=price&sortDescending=false&pageNumber=1&pageSize=10
```

### 4. Response Caching

Add response caching for improved performance on read operations:

```csharp
builder.Services.AddOutputCache(options =>
{
    options.AddBasePolicy(builder => builder.Cache());
    options.AddPolicy("ProductsList", builder => 
        builder.Cache()
               .Expire(TimeSpan.FromMinutes(5))
               .SetVaryByQuery("searchTerm", "category", "minPrice", "maxPrice", "sortBy", "sortDescending", "pageNumber", "pageSize"));
});

// In the app configuration
app.UseOutputCache();

// Apply to endpoint
app.MapGet("/api/products", async (IMediator mediator, ...) =>
{
    // Implementation
})
.CacheOutput("ProductsList");
```

### 5. Rate Limiting

Protect your API from abuse with rate limiting:

```csharp
using System.Threading.RateLimiting;

builder.Services.AddRateLimiter(options =>
{
    options.GlobalLimiter = PartitionedRateLimiter.Create<HttpContext, string>(context =>
        RateLimitPartition.GetFixedWindowLimiter(
            partitionKey: context.User.Identity?.Name ?? context.Request.Headers.Host.ToString(),
            factory: partition => new FixedWindowRateLimiterOptions
            {
                AutoReplenishment = true,
                PermitLimit = 100,
                QueueLimit = 0,
                Window = TimeSpan.FromMinutes(1)
            }));
});

// In the app configuration
app.UseRateLimiter();
```

---

## Conclusion

### Why Minimal API with CQRS?

Combining **Minimal APIs** with the **CQRS pattern** provides a powerful, scalable architecture for modern .NET applications:

#### Performance Benefits
- **Faster Startup**: Minimal APIs reduce startup time compared to traditional MVC
- **Lower Memory Footprint**: Less overhead from framework abstractions
- **Optimized Data Access**: Dapper provides near-native SQL performance
- **Independent Scaling**: Scale read and write operations separately

#### Development Benefits
- **Less Boilerplate**: Minimal code to achieve maximum functionality
- **Clear Separation**: Commands and queries are distinct and focused
- **Easy Testing**: Well-defined boundaries make unit testing straightforward
- **Maintainability**: Single Responsibility Principle applied throughout

#### Scalability Benefits
- **Read/Write Segregation**: Optimize each side independently
- **Caching Friendly**: Easy to cache query results without affecting commands
- **Microservices Ready**: Perfect for breaking down into smaller services
- **Cloud Native**: Lightweight containers with fast cold starts

#### Best Use Cases
1. **Microservices Architecture**: Lightweight services that do one thing well
2. **API-First Applications**: Backend APIs for mobile and web applications
3. **High-Performance Systems**: Applications requiring maximum throughput
4. **Event-Driven Systems**: Easily integrate with message queues and event buses
5. **Serverless Deployments**: Azure Functions, AWS Lambda compatibility

### Next Steps

1. **Add Integration Tests**: Test the entire flow from HTTP request to database
2. **Implement Domain Events**: Use events for cross-aggregate communication
3. **Add API Versioning**: Support multiple API versions simultaneously
4. **Implement Health Checks**: Monitor database and external dependencies
5. **Add Observability**: Integrate with Application Insights or similar
6. **Container Deployment**: Create Docker images for easy deployment

### Additional Resources

- [ASP.NET Core Minimal APIs Documentation](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis)
- [MediatR Documentation](https://github.com/jbogard/MediatR)
- [Dapper Documentation](https://github.com/DapperLib/Dapper)
- [CQRS Pattern by Martin Fowler](https://martinfowler.com/bliki/CQRS.html)
- [Microsoft Identity Platform](https://docs.microsoft.com/en-us/azure/active-directory/develop/)

---

**Congratulations!** You now have a complete guide to building a production-ready Minimal API with CQRS in .NET 6. This architecture provides a solid foundation for scalable, maintainable, and high-performance applications.
