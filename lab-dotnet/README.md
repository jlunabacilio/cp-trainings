# REST in ASP.NET core

## How to create the template


1. Create a folder named users, get in and create a using template:

```
mkdir UsersBackend && cd UsersBackend
dotnet new webapi -controllers -f net8.
```

2. Check it out the files created:


| Name               | Description                                                                 |
|--------------------|-----------------------------------------------------------------------------|
| Controllers/       | Contains classes with public methods exposed as HTTP endpoints.            |
| Program.cs         | Configures services and the app's HTTP request pipeline, and contains the app's managed entrypoint. |
| UsersBackend.csproj| Contains configuration metadata for the project.                           |
| UsersBackend.http  | Contains configuration to test REST APIs directly from Visual Studio Code. |


3. Build and test the application using: `dotnet run`

```
4. Open in navigator https://localhost:{PORT}/weatherforecast
```
## Start Building

## Creating a new Model

1. Remove `WeatherForecast.cs` file.


2. Create a new model using dotnet CLI

```sh
cd UsersBackend &&
mkdir Models &&
dotnet new class -n User -o Models
```

3. In `User.cs`, paste next code starting in line 1:

```cs
namespace UsersBackend.Models ; //add .Models after users if something goes wrong
// for public class add id, username, password, email, role and department properties
public class User
```
4. Wait for GitHub Copilot to get suggestions and ajust as needed.

```cs
namespace UsersBackend.Models
{
    // Public class with properties: Id, Username, Email, Role, and Department
    public class User
    {
        public int Id { get; set; }
        public string? Username { get; set; }
        public string? Email { get; set; }
        public string? Role { get; set; }
        public string? Department { get; set; }
    }
}
```

## Creating a new Service


1. Create a new Service using dotnet CLI

```sh
mkdir Services
dotnet new class -n UserService -o Services
```

2. Open UserService.cs, it should looks like:

```cs
using UsersBackend.Models;

namespace UsersBackend.Services
{
    // I want to create an in-memory list of users
    // Create two static users
    // for list of two users I want to generate its id, username, email, role and department
    public static class UserService
    {
        // Static list of users
        static List<User> Users { get; }
        //static int nextId = 3; only for post method

        // Static constructor to initialize the list of users
        static UserService()
        {
            Users = new List<User>
            {
                new User { Id = 1, Username = "jluna", Email = "jluna@sample.com", Role = "Admin", Department = "IT" },
                new User { Id = 2, Username = "jdoe", Email = "jdoe@sample.com", Role = "User", Department = "HR" }
            };
        }
        // Method to get all users
        public static List<User> GetAllUsers()
        {
            return Users;
        }
    }

}

```
## Creating Controller


1. Create a new file in `Controllers/` path: `touch UsersController.cs`


2. Add next to controller file:

```cs
// Create a User controller to handle requests
using UsersBackend.Models;
using UsersBackend.Services;
using Microsoft.AspNetCore.Mvc; // Framework for calling API

namespace UsersBackend.Controllers
{
    [ApiController]
    [Route("[controller]")]
    public class UserController : ControllerBase
    {
        // Method to get all users
        [HttpGet]
        public ActionResult<List<User>> Get() => UserService.GetAll();

        // Method to get user by ID
        
        // Method to add a new user
    
        // Method to update an existing user

        // Method to delete a user
    
    }
}
```
Start accepting suggestions from Copilot to complete the controller.

## Enable connection to specific port and host


1. In GitHub Copilot Chat we can ask how to enable communication to specific host
and port, response should looks like in `Program.cs`:

```cs
app.UseCors(policy => policy
    .AllowAnyOrigin()
    .AllowAnyMethod()
    .AllowAnyHeader());
```
Full code:

```cs
var builder = WebApplication.CreateBuilder(args);

// Add services to the container.
builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();

// Configure the HTTP request pipeline.
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();

// Allow CORS to open port to connect an Angular app to the .NET Core app
app.UseCors(policy => policy
    .AllowAnyOrigin()
    .AllowAnyMethod()
    .AllowAnyHeader());

app.UseAuthorization();

app.MapControllers();

app.Run();
```
# Tests in ASP.NET


1. Create a separate project named UsersBackend.Tests for example.

```
mkdir UsersBackend.Tests
```

2. Move into the create folder and run:

```sh
cd UsersBackend.Tests
dotnet new mstest
```

3. Ask to the GitHub Copilot chat providing the existing Controller as a context,
for a unit test to validate when returning values, for example:

```
@workspace /tests Generate unit test for selected code
```
You can also ask: "create test cases: Verify the number of weather forecasts returned,
Verify the content of the weather forecasts".

It will return a similar response, that you should paste into the UnitTest1.cs:

```cs
using Microsoft.Extensions.Logging;
using Moq;
using UsersBackend.Controllers;
using Xunit;
using System.Linq;

namespace UsersBackend.Tests
{
    public class WeatherForecastControllerTests
    {
        private readonly WeatherForecastController _controller;
        private readonly Mock<ILogger<WeatherForecastController>> _loggerMock;

        public WeatherForecastControllerTests()
        {
            _loggerMock = new Mock<ILogger<WeatherForecastController>>();
            _controller = new WeatherForecastController(_loggerMock.Object);
        }

        [Fact]
        public void Get_ReturnsExpectedNumberOfWeatherForecasts()
        {
            // Act
            var result = _controller.Get();

            // Assert
            Assert.NotNull(result);
            Assert.Equal(5, result.Count());
        }

        [Fact]
        public void Get_ReturnsValidWeatherForecasts()
        {
            // Act
            var result = _controller.Get();

            // Assert
            Assert.All(result, forecast =>
            {
                Assert.InRange(forecast.TemperatureC, -100, 100);
                Assert.False(string.IsNullOrEmpty(forecast.Summary));
            });
        }
    }
}
```
3. Once pasted, in the terminal, run
```sh
dotnet test
```

4. You could see next error in the console:
```sh
/UsersBackend.Tests/UnitTest1.cs(1,17): error CS0234: The type or namespace
name 'Extensions' does not exist in the namespace 'Microsoft' (are you missing
an assembly reference?) [ /UsersBackend.Tests/UsersBackend.Tests.csproj]
/UsersBackend.Tests/UnitTest1.cs(2,7): error CS0246: The type or namespace name
'Moq' could not be found (are you missing a using directive or an assembly
reference?) [ /UsersBackend.Tests/UsersBackend.Tests.csproj]
/UsersBackend.Tests/UnitTest1.cs(3,20): error CS0234: The type or namespace
name 'Controllers' does not exist in the namespace 'UsersBackend' (are you
missing an assembly reference?) [
/UsersBackend.Tests/UsersBackend.Tests.csproj]
/UsersBackend.Tests/UnitTest1.cs(4,7): error CS0246: The type or namespace name
'Xunit' could not be found (are you missing a using directive or an assembly
reference?) [ /UsersBackend.Tests/UsersBackend.Tests.csproj]
/UsersBackend.Tests/UnitTest1.cs(10,26): error CS0246: The type or namespace
name 'WeatherForecastController' could not be found (are you missing a using
directive or an assembly reference?) [
/UsersBackend.Tests/UsersBackend.Tests.csproj]
/UsersBackend.Tests/UnitTest1.cs(11,26): error CS0246: The type or namespace
name 'Mock<>' could not be found (are you missing a using directive or an
assembly reference?) [ /UsersBackend.Tests/UsersBackend.Tests.csproj]
/UsersBackend.Tests/UnitTest1.cs(11,31): error CS0246: The type or namespace
name 'ILogger<>' could not be found (are you missing a using directive or an
assembly reference?) [ /UsersBackend.Tests/UsersBackend.Tests.csproj]
/UsersBackend.Tests/UnitTest1.cs(11,39): error CS0246: The type or namespace
name 'WeatherForecastController' could not be found (are you missing a using
directive or an assembly reference?) [
/UsersBackend.Tests/UsersBackend.Tests.csproj]
/UsersBackend.Tests/UnitTest1.cs(19,10): error CS0246: The type or namespace
name 'FactAttribute' could not be found (are you missing a using directive or
an assembly reference?) [ /UsersBackend.Tests/UsersBackend.Tests.csproj]
/UsersBackend.Tests/UnitTest1.cs(19,10): error CS0246: The type or namespace
name 'Fact' could not be found (are you missing a using directive or an
assembly reference?) [ /UsersBackend.Tests/UsersBackend.Tests.csproj]
```
That means that you need to install te required packages.


5. Ask to the Chat again how to fix that error:

```sh
dotnet add package Microsoft.Extensions.Logging
dotnet add package Moq
dotnet add package xunit
dotnet add package xunit.runner.visualstudio
dotnet add reference ../UsersBackend/UsersBackend.csproj
```
```
dotnet restore
```


6. Run again dotnet test and that's it!

NOTE: If you start getting next error: UsersBackend.Tests/UnitTest1.cs(38,13): error
CS0104: 'Assert' is an ambiguous reference between
'Microsoft.VisualStudio.TestTools.UnitTesting.Assert' and 'Xunit.Assert'
[UsersBackend.Tests/UsersBackend.Tests.csproj] you can also ask to the Copilot chat to solve it.


