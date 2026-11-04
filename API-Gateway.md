# Configure and Evaluate API Gateway Routing Using YARP

<img width="1800" height="900" alt="00-api-gateway" src="https://github.com/user-attachments/assets/e048e620-299a-4261-a2ce-5ea3a3c23b1e" />


## Create the Gateway Project

The gateway needs no controllers or database, so we use the smallest template, `web`. Open a terminal in your `source` folder. If your path contains spaces, keep the quotes:

```bash
cd "path\to\Leave-Management-System\source"
dotnet new web -n ApiGateway -o ApiGateway
```

A new `ApiGateway` folder appears next to the three services.

## Add the YARP Package

```bash
cd ApiGateway
dotnet add package Yarp.ReverseProxy
dotnet build
```

---

## Write the Routes

This is where routing happens. Replace everything in `ApiGateway/appsettings.json`:

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*",
  "ReverseProxy": {
    "Routes": {
      "employee-route": {
        "ClusterId": "employee-cluster",
        "Match": { "Path": "/employee-service/{**catch-all}" },
        "Transforms": [{ "PathRemovePrefix": "/employee-service" }]
      },
      "leavetype-route": {
        "ClusterId": "leavetype-cluster",
        "Match": { "Path": "/leavetype-service/{**catch-all}" },
        "Transforms": [{ "PathRemovePrefix": "/leavetype-service" }]
      },
      "leaverequest-route": {
        "ClusterId": "leaverequest-cluster",
        "Match": { "Path": "/leaverequest-service/{**catch-all}" },
        "Transforms": [{ "PathRemovePrefix": "/leaverequest-service" }]
      }
    },
    "Clusters": {
      "employee-cluster": {
        "Destinations": {
          "employee-service": { "Address": "http://localhost:5101/" }
        }
      },
      "leavetype-cluster": {
        "Destinations": {
          "leavetype-service": { "Address": "http://localhost:5102/" }
        }
      },
      "leaverequest-cluster": {
        "Destinations": {
          "leaverequest-service": { "Address": "http://localhost:5103/" }
        }
      }
    }
  }
}
```

| Setting                               | Meaning                                                                         |
| ------------------------------------- | ------------------------------------------------------------------------------- |
| `Routes`                              | Rules that match incoming requests                                              |
| `ClusterId`                           | Which cluster a matching request goes to                                        |
| `Match.Path`                          | The URL pattern to catch. `{**catch-all}` means "everything after this point"   |
| `Transforms`                          | Changes applied before forwarding. `PathRemovePrefix` strips the service prefix |
| `Clusters` and `Destinations.Address` | The real service address                                                        |

**Worked example:** `/employee-service/api/employees/1` matches `employee-route`. The prefix is removed, leaving `/api/employees/1`, and the request goes to `http://localhost:5101/api/employees/1`.

The JSON must be valid. A missing comma or brace makes the gateway fail at startup.

## Write the Startup Code

```csharp
var builder = WebApplication.CreateBuilder(args);

// Register YARP and load the routes and clusters from appsettings.json
builder.Services.AddReverseProxy()
    .LoadFromConfig(builder.Configuration.GetSection("ReverseProxy"));

var app = builder.Build();

app.MapGet("/", () => "API Gateway is running");

// Turn on the routing rules
app.MapReverseProxy();

app.Run();
```

## Set the Gateway's Port

In `ApiGateway/Properties/launchSettings.json`, in the profile named **`http`**, set only this line:

```json
"applicationUrl": "http://localhost:5100",
```

---

## Set Up All Endpoint For Testing in Scalar For APIGateWay

```csharp
using Scalar.AspNetCore;

var builder = WebApplication.CreateBuilder(args);
builder.Services.AddOpenApi();
// Register YARP and load the routes and clusters from appsettings.json
builder.Services.AddReverseProxy()
    .LoadFromConfig(builder.Configuration.GetSection("ReverseProxy"));

var app = builder.Build();


// Test pages are for development only
if (app.Environment.IsDevelopment())
{
    app.MapScalarApiReference("/docs/employee", options => options
        .AddDocument("v1", "Employee Service", "/employee-service/openapi/v1.json")
        .AddServer("http://localhost:5100/employee-service", "Through the gateway"));

    app.MapScalarApiReference("/docs/leavetype", options => options
        .AddDocument("v1", "LeaveType Service", "/leavetype-service/openapi/v1.json")
        .AddServer("http://localhost:5100/leavetype-service", "Through the gateway"));

    app.MapScalarApiReference("/docs/leaverequest", options => options
        .AddDocument("v1", "LeaveRequest Service", "/leaverequest-service/openapi/v1.json")
        .AddServer("http://localhost:5100/leaverequest-service", "Through the gateway"));
}

// Forward requests to the backend services
app.MapReverseProxy();

app.Run();

```

| Part                   | Simple meaning                                                                                                  |
| ---------------------- | --------------------------------------------------------------------------------------------------------------- |
| `"/docs/leaverequest"` | The page address. You open `http://localhost:5100/docs/leaverequest`                                            |
| `AddDocument(...)`     | **What to show.** The list of LeaveRequest endpoints, fetched from the service **through the gateway**          |
| `AddServer(...)`       | **Where to send tests.** Every **Test Request** goes to the gateway, and the gateway forwards it to the service |

## Test APIGateWay

### In a employee-service Project Properties/launchSetting.Json

```csharp
{
  "$schema": "https://json.schemastore.org/launchsettings.json",
  "profiles": {
    "http": {
      "commandName": "Project",
      "dotnetRunMessages": true,
      "launchBrowser": false, // Disabled because all API requests and documentation are accessed through the API Gateway, which acts as the single entry point for all microservices.
      "launchUrl": "scalar",
      "applicationUrl": "http://localhost:5101",
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      }
    },
    "https": {
      "commandName": "Project",
      "dotnetRunMessages": true,
      "launchBrowser": false,// Disabled because all API requests and documentation are accessed through the API Gateway, which acts as the single entry point for all microservices.
      "launchUrl": "scalar",
      "applicationUrl": "https://localhost:7072;http://localhost:5101",
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      }
    }
  }
}
```

### Same For leavetype & leaverequest Services Properties/launchSetting.Json
