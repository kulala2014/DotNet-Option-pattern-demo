# DotNet-Option-pattern-demo
This repo is used to learn option pattern in .net core.

### 👇 IConfiguration
IConfiguration is a simple and lightweight approach for loading configurations in ASP.NET Core applications from the appsettings file.However, we need a more robust solution for handling a bit more advanced requirements like Validations, Type-Safety, and Reloading. This is where the Options Pattern comes in.
Here are the other cons of using IConfiguration in ASP.NET Core applications:
<ul>
  <li>IConfiguration to load the required configurations from the appsettings.json. This can be bad for the security of the application since IConfigration has access to all the other configuration options as well which are not meant to be derived. You will also have to name your keys properly to avoid runtime errors.</li>
  <li><strong>No Validation – </strong>IConfiguration does not perform any validation over the configuration values, which might be fatal during the application runtime.</li>
  <li><strong>No Type-Safety - </strong>As mentioned earlier, the interface reads configurations as strings that have to be parsed manually.This also increases the chances of configuration-related issues.</li>
  <li><strong>No Default values – </strong>In case the required key is empty / not found in appsettings, there is no built-in way to return a default value that can be used throughout the application.</li>
  <li><strong>No Type-Safety - </strong>As mentioned earlier, the interface reads configurations as strings that have to be parsed manually.This also increases the chances of configuration-related issues.</li>
</ul>
Use below sample to show the option pattern:
appsettings.json

```
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*",
  "WeatherOptions": {
    "City": "Trivandrum",
    "State": "Kerala1",
    "Temperature": 99,
    "Summary": "Warm"
  }
}
```
Use WeathersController to read settings from appsettings.json with IConfiguration:

```
    [Route("[controller]")]
    [ApiController]
    public class WeathersController : ControllerBase
    {
        private readonly IConfiguration _configuration;
        /// <param name="configuration"></param>
        /// <param name="weatherOptions"></param>
        public WeathersController(IConfiguration configuration)
        {
            _configuration = configuration;
        }

        /// <summary>
        /// IConfiguration to load the required configurations from the appsettings.json. This can be bad for the security of the application since IConfigration has access to all the other configuration options as well which are not meant to be derived. You will also have to name your keys properly to avoid runtime errors.
        ///Here are the other cons of using IConfiguration in ASP.NET Core applications.
        ///No Validation: IConfiguration does not perform any validation over the configuration values, which might be fatal during the application runtime.
        ///No Type-Safety: As mentioned earlier, the interface reads configurations as strings that have to be parsed manually.This also increases the chances of configuration-related issues.
        ///No Default values: In case the required key is empty / not found in appsettings, there is no built-in way to return a default value that can be used throughout the application.
        ///IConfiguration is a simple and lightweight approach for loading configurations in ASP.NET Core applications from the appsettings file.However, we need a more robust solution for handling a bit more advanced requirements like Validations, Type-Safety, and Reloading. This is where the Options Pattern comes in.
        /// </summary>
        /// <returns></returns>
        [HttpGet("config")]
        public IActionResult Get()
        {
            var city = _configuration.GetValue<string>("WeatherOptions:City");
            var state = _configuration.GetValue<string>("WeatherOptions:State");
            var temperature = _configuration.GetValue<string>("WeatherOptions:Temperature");
            var summary = _configuration.GetValue<string>("WeatherOptions:Summary");
            return Ok(new
            {
                City = city,
                State = state,
                Temperature = temperature,
                Summary = summary
            });
        }
}
```
### 👇 Options
The options pattern uses classes to provide strongly typed access to groups of related settings. When configuration settings are isolated by scenario into separate classes, the app adheres to two important software engineering principles:

#### 👇 RequestEncapsulation:
Classes that depend on configuration settings depend only on the configuration settings that they use.
#### 👇 Separation of Concerns:
Settings for different parts of the app aren't dependent or coupled to one another.
Options also provide a mechanism to validate configuration data. For more information, see the Options validation section.

And Options have the below three types of interfaces to load settings:
### 👇 IOptions:
The IOptions interface load the configuration values only once, during the application startup.
### 👇 IOptionsSnapshot:
IOptionsSnapshot - this is a scoped service that gives a snapshot of options at the time the constructor is invoked.
### 👇 IOptionsMonitor:
IOptionsMonitor - this is a singleton service that gets the current value at any time

The major difference is the lifetime of these instances:
IOptionsMonitor is registered as Singleton, whereas the IOptionsSnapshot is registered as Scoped.
#### 👇 When to use IOptions, IOptionsMonitor, and IOptionsSnapshot?
<ul>
  <li>Prefer to use IOptions, when you are not expecting your configuration values to change.</li>
  <li>Use IOptionsSnapshot when you expect your values to change, but want them to be uniform for the entire request cycle.</li>
  <li>Use IOptionsMonitor when you need real-time options data</li>
</ul>

For example, AddOptions in startup:
```
using LearningOptions.Models;
using Microsoft.EntityFrameworkCore;

var builder = WebApplication.CreateBuilder(args);

// Add services to the container.

builder.Services.AddControllers();
builder.Services.AddOptions<WeatherOptions>().BindConfiguration(nameof(WeatherOptions))
    .ValidateDataAnnotations()
    .ValidateOnStart()
    .Validate(options =>
    {
        if (options.State != "Kerala1") return false;
        return true;
    });
builder.Services.AddDbContext<TodoContext>(opt => 
    opt.UseInMemoryDatabase("TodoList"));
builder.Services.AddEndpointsApiExplorer();

var app = builder.Build();

// Configure the HTTP request pipeline.

app.UseAuthorization();

app.MapControllers();

app.Run();
```

And use IOptions, IOptionsSnapshot, IOptionsMonitor, IConfiguration in one controller:
```
    [Route("[controller]")]
    [ApiController]
    public class WeatherController : ControllerBase
    {
        private readonly IConfiguration _configuration;
        private readonly WeatherOptions _weatherOptions;
        private readonly WeatherOptions _optionsSnapshot;
        private readonly WeatherOptions _optionsMonitor;
        /// <summary>
        /// The IOptions interface load the configuration values only once, during the application startup.
        /// If we want to always load the latest value from appsettings.json, we could use two following interfaces:
        /// IOptionsSnapshot - this is a scoped service that gives a snapshot of options at the time the constructor is invoked.
        /// IOptionsMonitor - this is a singleton service that gets the current value at any time
        /// </summary>
        /// <param name="configuration"></param>
        /// <param name="weatherOptions"></param>
        public WeatherController(IConfiguration configuration, IOptions<WeatherOptions> weatherOptions, IOptionsSnapshot<WeatherOptions> optionsSnapshot, IOptionsMonitor<WeatherOptions> optionsMonitor)
        {
            _configuration = configuration;
            _weatherOptions = weatherOptions.Value;
            _optionsSnapshot = optionsSnapshot.Value;
            _optionsMonitor = optionsMonitor.CurrentValue;
        }

        /// <summary>
        /// IConfiguration to load the required configurations from the appsettings.json. This can be bad for the security of the application since IConfigration has access to all the other configuration options as well which are not meant to be derived. You will also have to name your keys properly to avoid runtime errors.
        ///Here are the other cons of using IConfiguration in ASP.NET Core applications.
        ///No Validation: IConfiguration does not perform any validation over the configuration values, which might be fatal during the application runtime.
        ///No Type-Safety: As mentioned earlier, the interface reads configurations as strings that have to be parsed manually.This also increases the chances of configuration-related issues.
        ///No Default values: In case the required key is empty / not found in appsettings, there is no built-in way to return a default value that can be used throughout the application.
        ///IConfiguration is a simple and lightweight approach for loading configurations in ASP.NET Core applications from the appsettings file.However, we need a more robust solution for handling a bit more advanced requirements like Validations, Type-Safety, and Reloading. This is where the Options Pattern comes in.
        /// </summary>
        /// <returns></returns>
        [HttpGet("config")]
        public IActionResult Get()
        {
            var city = _configuration.GetValue<string>("WeatherOptions:City");
            var state = _configuration.GetValue<string>("WeatherOptions:State");
            var temperature = _configuration.GetValue<string>("WeatherOptions:Temperature");
            var summary = _configuration.GetValue<string>("WeatherOptions:Summary");
            return Ok(new
            {
                City = city,
                State = state,
                Temperature = temperature,
                Summary = summary
            });
        }

        /// <summary>
        /// The major difference is the lifetime of these instances:
        /// IOptionsMonitor is registered as Singleton, 
        /// whereas the IOptionsSnapshot is registered as Scoped.
        ///When to use IOptions, IOptionsMonitor, and IOptionsSnapshot?
        ///Prefer to use IOptions, when you are not expecting your configuration values to change.
        ///Use IOptionsSnapshot when you expect your values to change, but want them to be uniform for the entire request cycle.
        ///Use IOptionsMonitor when you need real-time options data
        /// </summary>
        /// <returns></returns>
        [HttpGet("options")]
        public IActionResult GetFromOptionsPattern()
        {
            var response = new
            {
                options = new { _weatherOptions.City, _weatherOptions.State, _weatherOptions.Temperature, _weatherOptions.Summary },
                optionsSnapshot = new { _optionsSnapshot.City, _optionsSnapshot.State, _optionsSnapshot.Temperature, _optionsSnapshot.Summary },
                optionsMonitor = new { _optionsMonitor.City, _optionsMonitor.State, _optionsMonitor.Temperature, _optionsMonitor.Summary }
            };
            return Ok(response);
        }
    }
```

