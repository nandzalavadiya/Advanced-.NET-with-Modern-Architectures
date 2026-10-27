# Lab 7: Develop a Basic .NET Microservice with CRUD Operations


<img width="1800" height="840" alt="00-crud-request-flow" src="https://github.com/user-attachments/assets/ab76f096-a1a8-41c3-a4f6-cafae5abcd11" />


---

## Step 1: Install the Packages

```bash
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
dotnet add package Microsoft.EntityFrameworkCore.Tools
dotnet add package Scalar.AspNetCore
```

| Package                         | What it does                               |
| ------------------------------- | ------------------------------------------ |
| `EntityFrameworkCore.SqlServer` | Lets EF Core talk to SQL Server            |
| `EntityFrameworkCore.Tools`     | Adds the migration commands used in Step 6 |
| `Scalar.AspNetCore`             | Draws an interactive page to test the API  |

---

## Step 2: Create the Model

Create a folder `Models`, then add `Models/LeaveType.cs`:

```csharp
using System.ComponentModel.DataAnnotations;

namespace LeaveTypeService.Models;

public class LeaveType
{
    public int Id { get; set; }

    [Required]
    [MaxLength(100)]
    public string Name { get; set; } = string.Empty;

    public string Description { get; set; } = string.Empty;

    [Range(1, 365)]
    public int MaxDaysPerYear { get; set; }
}
```

---

## Step 3: Create the Database Context

The context is EF Core's link between your classes and the database.

Create a folder `Data`, then add `Data/AppDbContext.cs`:

```csharp
using LeaveTypeService.Models;
using Microsoft.EntityFrameworkCore;

namespace LeaveTypeService.Data;

public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options)
    {
    }

    public DbSet<LeaveType> LeaveTypes => Set<LeaveType>();
}
```

`DbSet<LeaveType> LeaveTypes` stands for the `LeaveTypes` table. You add, read, update, and delete rows through it.

---

## Step 4: Add the Connection String

In `appsettings.json`, add the `ConnectionStrings` block and keep the other sections:

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*",
  "ConnectionStrings": {
    "ConnectionString": "Data Source=NAND-07;Initial Catalog=LeaveTypeDB;Integrated Security=True;TrustServerCertificate=True;"
  }
}
```

---

## Step 5: Register the Database and Scalar

Replace everything in `Program.cs`:

```csharp
using LeaveTypeService.Data;
using Microsoft.EntityFrameworkCore;
using Scalar.AspNetCore;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();

// Generates the OpenAPI document that Scalar reads
builder.Services.AddOpenApi();

builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("ConnectionString")));

var app = builder.Build();

// API documentation is for development only
if (app.Environment.IsDevelopment())
{
    app.MapOpenApi();
    app.MapScalarApiReference();
}

app.MapGet("/", () => "Leave Type Service is running");

app.MapGet("/health", () => Results.Ok(new
{
    service = "LeaveTypeService",
    status = "Running",
    responsibility = "Manages leave categories such as Casual, Sick and Earned leave"
}));

app.MapControllers();

app.Run();
```

---

## Step 6: Create the Database with a Migration

A **migration** is a script EF Core generates by comparing your model with the database. Running it creates the database and the `LeaveTypes` table.

```powershell
Add-Migration InitialCreate
```

```powershell
Update-Database
```

---

## Step 7: Write the Controller

A controller receives HTTP requests and decides what to return. Add `Controllers/LeaveTypesController.cs`:

```csharp
using LeaveTypeService.Data;
using LeaveTypeService.Models;
using Microsoft.EntityFrameworkCore;

namespace LeaveTypeService.Controllers;

[ApiController]
[Route("api/[controller]")]
public class LeaveTypesController : ControllerBase
{
    private readonly AppDbContext _db;

    public LeaveTypesController(AppDbContext db)
    {
        _db = db;
    }

    // READ ALL: GET /api/leavetypes
    [HttpGet]
    public async Task<ActionResult<List<LeaveType>>> GetAll()
    {
        return await _db.LeaveTypes.ToListAsync();
    }

    // READ ONE: GET /api/leavetypes/1
    [HttpGet("{id}")]
    public async Task<ActionResult<LeaveType>> GetById(int id)
    {
        var item = await _db.LeaveTypes.FindAsync(id);
        if (item is null)
        {
            return NotFound();
        }
        return item;
    }

    // CREATE: POST /api/leavetypes
    [HttpPost]
    public async Task<ActionResult<LeaveType>> Create(LeaveType leaveType)
    {
        leaveType.Id = 0; // the database assigns the Id, ignore any value sent by the client
        _db.LeaveTypes.Add(leaveType);
        await _db.SaveChangesAsync();
        return CreatedAtAction(nameof(GetById), new { id = leaveType.Id }, leaveType);
    }

    // UPDATE: PUT /api/leavetypes/1
    [HttpPut("{id}")]
    public async Task<IActionResult> Update(int id, LeaveType updated)
    {
        var existing = await _db.LeaveTypes.FindAsync(id);
        if (existing is null)
        {
            return NotFound();
        }

        existing.Name = updated.Name;
        existing.Description = updated.Description;
        existing.MaxDaysPerYear = updated.MaxDaysPerYear;

        await _db.SaveChangesAsync();
        return NoContent();
    }

    // DELETE: DELETE /api/leavetypes/1
    [HttpDelete("{id}")]
    public async Task<IActionResult> Delete(int id)
    {
        var existing = await _db.LeaveTypes.FindAsync(id);
        if (existing is null)
        {
            return NotFound();
        }

        _db.LeaveTypes.Remove(existing);
        await _db.SaveChangesAsync();
        return NoContent();
    }
}
```

---

## Step 8: Run and Test with Scalar

<img width="1767" height="857" alt="API Test GetAll" src="https://github.com/user-attachments/assets/4ab50b17-955f-4aa6-8dde-f22d1a42feb2" />

<img width="1740" height="841" alt="API Test POST" src="https://github.com/user-attachments/assets/a0f600c1-de96-4b5d-9e69-d443c51ffe20" />

