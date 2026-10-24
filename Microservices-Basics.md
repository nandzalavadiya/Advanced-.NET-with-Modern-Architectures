# Microservices Concepts & Baseline .NET API Architecture

---


<img width="1800" height="960" alt="monolith-vs-microservices-1" src="https://github.com/user-attachments/assets/1fde29b1-ac44-4e03-a3ed-f6bee33b77bf" />


## What We Will Build

Three small ASP.NET Core Web API projects. Each runs on its own port and has two endpoints: `/` and `/health`.

| Service               | Responsibility                                       | Port |
| --------------------- | ---------------------------------------------------- | ---- |
| `EmployeeService`     | Manages employee information                         | 5101 |
| `LeaveTypeService`    | Manages leave categories (Casual, Sick, Earned)      | 5102 |
| `LeaveRequestService` | Manages leave applications and their approval status | 5103 |

---

## Concept in Simple Words

**Monolith:** one big application does everything. If one part fails, the whole application can go down.

**Microservices:** many small applications, each owning one business job. They run separately and talk to each other over HTTP or messages.

---

## Practical Workflow

```mermaid
flowchart TB
  Client["Client (Browser / Postman)"]
  Client --> E["EmployeeService<br/>http://localhost:5101"]
  Client --> L["LeaveTypeService<br/>http://localhost:5102"]
  Client --> R["LeaveRequestService<br/>http://localhost:5103"]
```

---

## Project Structure

<img width="676" height="1000" alt="Project Structure-1" src="https://github.com/user-attachments/assets/4a178368-2e9a-4c1c-8ad6-9e3dc0c83f44" />


---

## 7. Step-by-Step Implementation

### Step 1: Create the empty folders Leave-Management-System

### Step 2: Inside a Leave-Management-System Create SubFolder source (Leave-Management-System/source)

### Step 3: Create a new project → ASP.NET Core Web API

1. **Create a new project For Employee Services → ASP.NET Core Web API**.


   <img width="1896" height="1057" alt="Emp Services Creation-1" src="https://github.com/user-attachments/assets/3b99e518-3bc1-40a4-8386-8ccf91699ef9" />
   
<img width="1897" height="1062" alt="Emp Services Creation Step-2-1" src="https://github.com/user-attachments/assets/929c5e53-ce98-4ff8-a489-2e375aa74c1a" />




2. Click **Create**.

### Step 4: Remove the template sample code

Delete these files:

- `WeatherForecast.cs`
- `Controllers/WeatherForecastController.cs`
- `EmployeeService.http`

### Step 5: Replace `Program.cs`

**File:** `source/EmployeeService/EmployeeService/Program.cs`

```csharp
var builder = WebApplication.CreateBuilder(args);

// Controllers are enabled now, used from Lab 7 onward
builder.Services.AddControllers();

var app = builder.Build();

// Proves the service is alive
app.MapGet("/", () => "Employee Service is running");

// JSON response: shows what a real API reply looks like
app.MapGet("/health", () => Results.Ok(new
{
    service = "EmployeeService",
    status = "Running",
    responsibility = "Manages employee information"
}));

app.MapControllers();

app.Run();
```

### Step 6: Set a fixed port

**File:** `Properties/launchSettings.json`

In the profile named `"http"` & `"https"`, change `applicationUrl`:

```json
"launchBrowser": true,
"applicationUrl": "http://localhost:5101"

```

### Step 7: Run and verify EmployeeService


<img width="1907" height="1005" alt="Emp Services Run Part-1-1" src="https://github.com/user-attachments/assets/b10895bf-1336-4977-b86c-5cd0e3343554" />

<img width="1897" height="1062" alt="Emp Services Creation Step-2-1" src="https://github.com/user-attachments/assets/3d49c82b-4dfd-4787-a687-2e5e5e7bae3d" />

<img width="1933" height="1050" alt="Emp Services Run Part-3-1" src="https://github.com/user-attachments/assets/29b88c65-ced4-467f-8aea-5204da6195e7" />

### Step 8: Create LeaveTypeService and LeaveRequestService

<img width="1907" height="1037" alt="Nested Project Createtion-1" src="https://github.com/user-attachments/assets/d10942ec-dee3-4cda-b738-eea69e525c0d" />


**`LeaveTypeService/Program.cs`** (port **5102**)

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();

var app = builder.Build();

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

```json
"launchBrowser": true,
"applicationUrl": "http://localhost:5102"

```

**`LeaveRequestService/Program.cs`** (port **5103**)

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();

var app = builder.Build();

app.MapGet("/", () => "Leave Request Service is running");

app.MapGet("/health", () => Results.Ok(new
{
    service = "LeaveRequestService",
    status = "Running",
    responsibility = "Manages leave applications and their approval status"
}));

app.MapControllers();

app.Run();
```

```json
"launchBrowser": true,
"applicationUrl": "http://localhost:5103"

```

## Step 9 : Run and verify LeaveTypeService/LeaveRequestService

<img width="1897" height="1015" alt="Run(Select Which one Run)-1" src="https://github.com/user-attachments/assets/a6d717d9-93e3-42f4-b41e-0a0691614063" />


## 10. Run the All Services into Together

<img width="1910" height="1040" alt="Run All Services Together-1" src="https://github.com/user-attachments/assets/6628d908-54b9-4def-b894-2fde80319adf" />

<img width="1020" height="692" alt="Run All Services Together Part-2-1" src="https://github.com/user-attachments/assets/e9c3ab56-dce1-4704-aab6-c7923be048bf" />


