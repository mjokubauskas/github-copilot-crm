# Setup Documentation for Backend

## Introduction
This document serves as a comprehensive guide to setting up the backend of the CRM system using Minimal API, CQRS with MediatR, Entra ID Authentication, flexible filtering, fuzzy search, and sorting.

## Prerequisites
- [.NET SDK](https://dotnet.microsoft.com/download) (version 6.0 or above)
- NuGet packages:
  - MediatR
  - Microsoft.AspNetCore.Authentication
  - Microsoft.AspNetCore.Mvc

## Minimal API Setup
1. Create a new ASP.NET Core Web API project.
2. Define your API endpoints using the `MapGet`, `MapPost`, etc., methods in the `Program.cs` file.
3. Example:
   ```csharp
   var builder = WebApplication.CreateBuilder(args);
   var app = builder.Build();

   app.MapGet("/api/users", async () => {
       return await userService.GetAllUsersAsync();
   });
   ```

## CQRS with MediatR
1. Install the MediatR package via NuGet.
2. Create commands and queries as separate classes.
3. Use `IMediator` to send commands and queries.
4. Example:
   ```csharp
   public class GetUserQuery : IRequest<User>
   {
       public int UserId { get; set; }
   }
   ```

## Entra ID Authentication
1. Register your application with Entra ID.
2. Configure authentication in `Program.cs`:
   ```csharp
   builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
       .AddJwtBearer(options => {
           options.Authority = "https://login.microsoftonline.com/{tenantId}/";
           options.Audience = "{clientId}";
       });
   ```

## Flexible Filtering
1. Implement filtering logic in your query handlers.
2. Accept filter parameters in your API endpoint.
3. Example:
   ```csharp
   public async Task<IEnumerable<User>> GetUsersAsync(string? filter)
   {
       // Filtering logic
   }
   ```

## Fuzzy Search
1. Utilize a search library like Lucene.Net or implement custom logic.
2. Create a search service and integrate it with your queries.

## Sorting
1. Define sorting parameters in your API endpoints.
2. Example:
   ```csharp
   public async Task<IEnumerable<User>> GetUsersAsync(string? sortBy)
   {
       // Sorting logic
   }
   ```

## Conclusion
This guide provides a foundational approach to setting up a robust backend for the CRM system. For additional resources, refer to the official documentation for each technology used.