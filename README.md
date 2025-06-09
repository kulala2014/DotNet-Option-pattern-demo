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

```
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
```

### The options pattern uses classes to provide strongly typed access to groups of related settings. When configuration settings are isolated by scenario into separate classes, the app adheres to two important software engineering principles:

#### 👇 RequestEncapsulation:
Classes that depend on configuration settings depend only on the configuration settings that they use.
#### 👇 Separation of Concerns:
Settings for different parts of the app aren't dependent or coupled to one another.
Options also provide a mechanism to validate configuration data. For more information, see the Options validation section.
