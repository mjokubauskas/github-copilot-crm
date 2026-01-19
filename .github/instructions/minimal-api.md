# Setting Up a Minimal API with CQRS using .NET 6

## Introduction
Minimal API is a lightweight programming model in .NET 6 that allows developers to quickly create web APIs with minimal setup. By combining Minimal API with CQRS (Command Query Responsibility Segregation) using MediatR, and optionally adding Authentication using Microsoft Entra ID, you can create a robust and maintainable web API backend.

---

## Step 1: Create Minimal API project
- Install the .NET SDK 6 if not already installed.
  ```bash
  dotnet new web -n MinimalApiCqrs
  ```
- Install required NuGet packages:
  ```bash
  # CQRS and MediatR
  dotnet add package MediatR.Extensions.Microsoft.DependencyInjection

  # Authentication
  dotnet add package Microsoft.AspNetCore.Authentication.JwtBearer
  dotnet add package Microsoft.Identity.Web
  ```

---

## Step 2: Set up MediatR for Commands and Queries

1. **Create Commands**: For example, to create an entity `Task`:
   - Define a `CreateTaskCommand`:
     ```csharp
     public record CreateTaskCommand(string Title, string Description) : IRequest<int>;
     ```

   - Implement the `CreateTaskCommandHandler`:
     ```csharp
     public class CreateTaskCommandHandler : IRequestHandler<CreateTaskCommand, int>
     {
         // Inject dependencies
         private readonly IDbConnection _db;

         public CreateTaskCommandHandler(IDbConnection db)
         {
             _db = db;
         }

         public async Task<int> Handle(CreateTaskCommand request, CancellationToken cancellationToken)
         {
             var sql = "INSERT INTO Tasks (Title, Description) VALUES (@Title, @Description); SELECT CAST(SCOPE_IDENTITY() as int);";
             return await _db.QuerySingleAsync<int>(sql, new { request.Title, request.Description });
         }
     }
     ``

2. **Create Queries**:
   - Define a `GetTasksQuery`:
     ```csharp
     public record GetTasksQuery : IRequest<IEnumerable<TaskDto>>;
     ```
   - Implement `GetTasksQueryHandler`:
     ```csharp
     public class GetTasksQueryHandler : IRequestHandler<GetTasksQuery, IEnumerable<TaskDto>>
     {
         private readonly IDbConnection _db;

         public GetTasksQueryHandler(IDbConnection db)
         {
             _db = db;
         }

         public async Task<IEnumerable<TaskDto>> Handle(GetTasksQuery request, CancellationToken cancellationToken)
         {
             var sql = "SELECT * FROM Tasks;";
             return await _db.QueryAsync<TaskDto>(sql);
         }
     }
     ```
---

## Step 3: Configure Middleware and Endpoints
Update your `Program.cs` file to set up required middleware and minimal API endpoints using CQRS and MediatR.

```csharp
var builder = WebApplication.CreateBuilder(args);

// Register MediatR
builder.Services.AddMediatR(typeof(Program));

// Optional: Add Authentication (JWT or Entra ID)
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddMicrosoftIdentityWebApi(builder.Configuration.GetSection("AzureAd"));

var app = builder.Build();

// Authentication Middleware app.UseAuthentication(); app.UseAuthorization();

// Example: Minimal API for commands and queries
app.MapPost("/tasks", async (CreateTaskCommand command, IMediator mediator) => {
    var id = await mediator.Send(command);
    return Results.Created($"/tasks/{id}", new { Id = id });
});

app.MapGet("/tasks", async (IMediator mediator) => {
    var tasks = await mediator.Send(new GetTasksQuery());
    return Results.Ok(tasks);
});

app.Run();
```

---

## Step 4: Securing API with Microsoft Entra ID
To enable authentication:

1. Register your app in Azure AD or Entra ID portal.
2. Add the following configuration in `appsettings.json`:
   ```json
   {
       "AzureAd": {
           "Instance": "https://login.microsoftonline.com/",
           "Domain": "yourdomain.onmicrosoft.com",
           "TenantId": "your-tenant-id",
           "ClientId": "your-client-id",
           "Audience": "api://your-api-id",
           "CallbackPath": "/signin-oidc"
       }
   }
   ```
3. Add the `.AddMicrosoftIdentityWebApi()` method to enable authentication with Entra ID.
4. Restrict API endpoints using the `[Authorize]` attribute or policy-based authorization.

---

## Step 5: Add Swagger for API Documentation
To enable Swagger UI:
1. Add NuGet Package:
   ```bash
   dotnet add package Swashbuckle.AspNetCore
   ```
2. Configure Swagger in `Program.cs`:
   ```csharp
   builder.Services.AddEndpointsApiExplorer();
   builder.Services.AddSwaggerGen();

   app.UseSwagger();
   app.UseSwaggerUI();
   ```

Now you can navigate to `/swagger` to view your API documentation.

---

## Conclusion
Ready to leverage the lightweight and fast development process of Minimal APIs with the CQRS pattern in .NET? GitHub Copilot can assist you in generating and customizing the required code blocks, making development faster and error-free. Let GitHub Copilot be your coding assistant every step of the way!