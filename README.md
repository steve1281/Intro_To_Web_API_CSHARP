# Setup and Initial Configuration

Reference: [Intro to Web API](https://www.youtube.com/watch?v=87oOF9Ve-KA)
by *IAmTimCorey* (I am not associated with, but do reccomend)


## TIL
- you can generate a .gitignore file with: 
```
dotnet new gitignore
```


## Create a new project using Visual Studio
### screen 1
- C#, All platforms, Web
- ASP.NET Core Web API
### screen2
- APIDemo
- [ ] Place solution and project in the same directory
- C:\Users\steve\csharp_projects\ApiDemo
- APIDemoApp
### screen3
- defaults should be fine, we have .NET 8
- make sure HTTPS is enabled
- make sure OpenAPI support is enabled (newer word for Swagger)
- [Create]

##  Trial run
- Once Studio is launched, you can run it.
- It will open a little demo page at: https://localhost:7092/swagger/index.html
- This can be used to play with the default API microsoft provided
- Note that another application could call this; it doesn't have to be a web page.
- as an API, you would call: *https://localhost:7092/weatherforecast*
```
[{"date":"2025-07-09","temperatureC":-13,"temperatureF":9,"summary":"Bracing"},{"date":"2025-07-10","temperatureC":2,"temperatureF":35,"summary":"Balmy"},{"date":"2025-07-11","temperatureC":14,"temperatureF":57,"summary":"Bracing"},{"date":"2025-07-12","temperatureC":16,"temperatureF":60,"summary":"Cool"},{"date":"2025-07-13","temperatureC":24,"temperatureF":75,"summary":"Warm"}]
```
- this would need to be parsed by whoever called it.
- For example:
```
C:\Users\steve>curl https://localhost:7092/weatherforecast
[{"date":"2025-07-09","temperatureC":32,"temperatureF":89,"summary":"Scorching"},{"date":"2025-07-10","temperatureC":-1,"temperatureF":31,"summary":"Hot"},{"date":"2025-07-11","temperatureC":31,"temperatureF":87,"summary":"Sweltering"},{"date":"2025-07-12","temperatureC":33,"temperatureF":91,"summary":"Chilly"},{"date":"2025-07-13","temperatureC":-20,"temperatureF":-3,"summary":"Balmy"}]


```

## swagger.json file:

- application generates a file that can be used to create APIs
- https://localhost:7092/swagger/v1/swagger.json
```
{
  "openapi": "3.0.1",
  "info": {
    "title": "ApiDemo",
    "version": "1.0"
  },
  "paths": {
    "/WeatherForecast": {
      "get": {
        "tags": [
          "WeatherForecast"
        ],
        "operationId": "GetWeatherForecast",
        "responses": {
          "200": {
            "description": "OK",
            "content": {
              "text/plain": {
                "schema": {
                  "type": "array",
                  "items": {
                    "$ref": "#/components/schemas/WeatherForecast"
                  }
                }
              },
              "application/json": {
                "schema": {
                  "type": "array",
                  "items": {
                    "$ref": "#/components/schemas/WeatherForecast"
                  }
                }
              },
              "text/json": {
                "schema": {
                  "type": "array",
                  "items": {
                    "$ref": "#/components/schemas/WeatherForecast"
                  }
                }
              }
            }
          }
        }
      }
    }
  },
  "components": {
    "schemas": {
      "WeatherForecast": {
        "type": "object",
        "properties": {
          "date": {
            "type": "string",
            "format": "date"
          },
          "temperatureC": {
            "type": "integer",
            "format": "int32"
          },
          "temperatureF": {
            "type": "integer",
            "format": "int32",
            "readOnly": true
          },
          "summary": {
            "type": "string",
            "nullable": true
          }
        },
        "additionalProperties": false
      }
    }
  }
}
```

## Versioning
- for now, not that there is a ApiDemo v1
- you will be able to call supported versions
- generally, you start with version 1, and update with production releases

## Review Program.cs
- the Main looks like:
```
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

            app.UseAuthorization();


            app.MapControllers();

            app.Run();
```
- this is similair to other web apps; builder, add services, configure, run.
- note the if .. Is Development ; this comes from the launchSettings.json file.
- so, these launchSettings is only used locally - in production, they are not used, so not Development.
- remove the if ... clause if you want the API details available in production.
- note on MapControllers... it "looks for" controllers. 
- the example:
```
namespace ApiDemo.Controllers
{
    [ApiController]
    [Route("[controller]")]
    public class WeatherForecastController : ControllerBase
    {
        private static readonly string[] Summaries = new[]
        {
            "Freezing", "Bracing", "Chilly", "Cool", "Mild", "Warm", "Balmy", "Hot", "Sweltering", "Scorching"
        };

        private readonly ILogger<WeatherForecastController> _logger;

        public WeatherForecastController(ILogger<WeatherForecastController> logger)
        {
            _logger = logger;
        }

        [HttpGet(Name = "GetWeatherForecast")]
        public IEnumerable<WeatherForecast> Get()
        {
            return Enumerable.Range(1, 5).Select(index => new WeatherForecast
            {
                Date = DateOnly.FromDateTime(DateTime.Now.AddDays(index)),
                TemperatureC = Random.Shared.Next(-20, 55),
                Summary = Summaries[Random.Shared.Next(Summaries.Length)]
            })
            .ToArray();
        }
    }
}
```
- observe the tags
- observe the route
- observe the base class
- he prefers route starts with /api, so modify route to:
```
 [Route("api/[controller]")]
```
- note that the api is outside the square brakets.

## Model is WeatherForecast
- you can see the model in the return statement
- its generates some random stuff 
- the model is WeatherForecast, which is stored at the top of the project ? Weird.
```
namespace ApiDemo
{
    public class WeatherForecast
    {
        public DateOnly Date { get; set; }

        public int TemperatureC { get; set; }

        public int TemperatureF => 32 + (int)(TemperatureC / 0.5556);

        public string? Summary { get; set; }
    }
}
```
- I would have thought it would be in a Models sub-folder.
- actually, hang on a moment....
- make a folder Models
- move the file
- sure, let microsft have a go at fixing the namespacing
- and now its in a folder called Models, with a namespace called *ApiDemo.Models*

## git intermission
```
PS C:\Users\steve\csharp_projects\ApiDemo\ApiDemoApp> git init
Initialized empty Git repository in C:/Users/steve/csharp_projects/ApiDemo/ApiDemoApp/.git/

PS C:\Users\steve\csharp_projects\ApiDemo\ApiDemoApp> git status
On branch master

No commits yet

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        .gitignore
        ApiDemo/
        ApiDemoApp.sln

nothing added to commit but untracked files present (use "git add" to track)
PS C:\Users\steve\csharp_projects\ApiDemo\ApiDemoApp> git add .\.gitignore
PS C:\Users\steve\csharp_projects\ApiDemo\ApiDemoApp> git commit -m "add the ignore file"
[master (root-commit) 8a94931] add the ignore file
 1 file changed, 484 insertions(+)
 create mode 100644 .gitignore

PS C:\Users\steve\csharp_projects\ApiDemo\ApiDemoApp> git add .
PS C:\Users\steve\csharp_projects\ApiDemo\ApiDemoApp> git commit -m "setup, cleanup, ready to work... "
[master 6b92f27] setup, cleanup, ready to work...
 9 files changed, 184 insertions(+)
 create mode 100644 ApiDemo/ApiDemo.csproj
 create mode 100644 ApiDemo/ApiDemo.http
 create mode 100644 ApiDemo/Controllers/WeatherForecastController.cs
 create mode 100644 ApiDemo/Models/WeatherForecast.cs
 create mode 100644 ApiDemo/Program.cs
 create mode 100644 ApiDemo/Properties/launchSettings.json
 create mode 100644 ApiDemo/appsettings.Development.json
 create mode 100644 ApiDemo/appsettings.json
   
```

## Lets create our own controller
- right click Controller folder, Add : Controller
- DO NOT click MVC, click API instead
- choose API controller with Read/Write Actions (provides boilerplate)
- call it UsersController.cs
- produces this:
```
namespace ApiDemo.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    public class UsersController : ControllerBase
    {
        // GET: api/<UsersController>
        [HttpGet]
        public IEnumerable<string> Get()
        {
            return new string[] { "value1", "value2" };
        }

        // GET api/<UsersController>/5
        [HttpGet("{id}")]
        public string Get(int id)
        {
            return "value";
        }

        // POST api/<UsersController>
        [HttpPost]
        public void Post([FromBody] string value)
        {
        }

        // PUT api/<UsersController>/5
        [HttpPut("{id}")]
        public void Put(int id, [FromBody] string value)
        {
        }

        // DELETE api/<UsersController>/5
        [HttpDelete("{id}")]
        public void Delete(int id)
        {
        }
    }
}
```
- hey! how did it now to use api in the Route? That was a manual change. Clever.
- oh, not clever. api is the industry standard; the demo code was wrong. Sadness.
- GET/POST/PUT/DELETE are here.
- note that <UsersController> equates to Users in this case.
- observe {id} notation for passing parameters into the route.
- at this point, you can run and test your API. (go ahead, its pretty simple)
- he changes the get/{id} - so displays the "value {id}"
- he also has hot reload, so doesn't have to recompile. I don't.  Odd.
- I googled it.  There is either a project setting, or a missing package. Or both.
- In my case: Tools - Options - .NET/C++ Hot Reload: [x] Apply Hot Reload on File Save
- That was a nice diversion
- So now we have an API.

## git intermission
```
PS C:\Users\steve\csharp_projects\ApiDemo\ApiDemoApp> git status
On branch master
Untracked files:
  (use "git add <file>..." to include in what will be committed)
        ApiDemo/Controllers/UsersController.cs

nothing added to commit but untracked files present (use "git add" to track)
PS C:\Users\steve\csharp_projects\ApiDemo\ApiDemoApp> git add .
PS C:\Users\steve\csharp_projects\ApiDemo\ApiDemoApp> git commit -m "add a restful api"
[master ab284c8] add a restful api
 1 file changed, 42 insertions(+)
 create mode 100644 ApiDemo/Controllers/UsersController.c
```


## What about a minimal API?

- Create a new project in the solution right-click solotion, Add new project
- project type is ASP.NET Core Web Api
- call it MinimalApiDemo
- Uncheck [ ] Use controllers
- Generate code.
- Ok, open Program.cs. Observe that a whole bunch of stuff is globbered into one location.
- notice that app gets endpoints mapped on :
```
            app.MapGet("/weatherforecast", (HttpContext httpContext) =>
            {
                var forecast = Enumerable.Range(1, 5).Select(index =>
                    new WeatherForecast
                    {
                        Date = DateOnly.FromDateTime(DateTime.Now.AddDays(index)),
                        TemperatureC = Random.Shared.Next(-20, 55),
                        Summary = summaries[Random.Shared.Next(summaries.Length)]
                    })
                    .ToArray();
                return forecast;
            })
            .WithName("GetWeatherForecast")
            .WithOpenApi();
```
- My later version of the environment doesn't use a *record* like the course
- Set MinimalApiDemo as start up project
- Run

## git intermission
```
PS C:\Users\steve\csharp_projects\ApiDemo\ApiDemoApp> git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   ApiDemoApp.sln

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        MinimalApiDemo/

no changes added to commit (use "git add" and/or "git commit -a")
PS C:\Users\steve\csharp_projects\ApiDemo\ApiDemoApp> git add .
PS C:\Users\steve\csharp_projects\ApiDemo\ApiDemoApp> git commit -m "create minimal API"
[master 60d44de] create minimal API
 8 files changed, 151 insertions(+), 1 deletion(-)
 create mode 100644 MinimalApiDemo/MinimalApiDemo.csproj
 create mode 100644 MinimalApiDemo/MinimalApiDemo.http
 create mode 100644 MinimalApiDemo/Program.cs
 create mode 100644 MinimalApiDemo/Properties/launchSettings.json
 create mode 100644 MinimalApiDemo/WeatherForecast.cs
 create mode 100644 MinimalApiDemo/appsettings.Develoopment.json
 create mode 100644 MinimalApiDemo/appsettings.json
```
## Simplify into a really minimal API

- delete the WeatherForecast class. (in the minimal api project)
- modify Program.cs to bare bones:
```
namespace MinimalApiDemo
{
    public class Program
    {
        public static void Main(string[] args)
        {
            var builder = WebApplication.CreateBuilder(args);

            builder.Services.AddAuthorization();
            builder.Services.AddEndpointsApiExplorer();
            builder.Services.AddSwaggerGen();

            var app = builder.Build();

            app.UseSwagger();
            app.UseSwaggerUI();
            app.UseHttpsRedirection();
            app.UseAuthorization();

            app.MapGet("/testing", () => "Hello World");

            app.Run();
        }
    }
}

```
- run and test:
- check with simple curl:
```
C:\Users\steve>curl "https://localhost:7094/testing"
Hello World
```
- note, you can also do a MapPost, MapGet, MapDelete, MapPut

