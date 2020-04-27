Setting Up API Integration Tests With Autorization In .NET Core
==============================================
Recently I've introduced integration tests in of .NET Core projects. It's pretty simple as there' great support from Microsoft.AspNetCore.Mvc.Testing and Microsoft.AspNetCore.TestHost nuget packages.
I've used XUnit together with packages mentioned. In my opinion it works best due to ClassFixtures and Collections that XUnit is providing.

## Setting up Environment for Integration Tests
As I mentioned in previous section, I am using XUnit framework. It lets me create TestFixtures.

Test Fixtures are useful when you want to have different kind for sets, for example:
- Tests using unauthenticated user
- Tests using authenticated user
- Tests using user that has no authorization for certain resource
- Tests that check multitenancy mechanism
and so on.

```csharp
public class TestFixture : IClassFixture<WebApplicationFactory<Startup>>
{
    protected readonly WebApplicationFactory<Startup> Factory;
    protected HttpClient Client;

    public TestFixture(WebApplicationFactory<Startup> factory)
    {
        Factory = factory;
        SetupClient();
    }

    protected static HttpContent ConvertToHttpContent<T>(T data)
    {
        var jsonQuery = JsonConvert.SerializeObject(data);
        HttpContent httpContent = new StringContent(jsonQuery, Encoding.UTF8);
        httpContent.Headers.Remove("content-type");
        httpContent.Headers.Add("content-type", "application/json; charset=utf-8");

        return httpContent;
    }

    private void SetupClient()
    {
        Client = Factory.WithWebHostBuilder(builder =>
        {
            /* I am telling builder to use appsettings.Integration,
            that is correctly set up in my DevOps pipeline */
            builder.UseEnvironment("Integration");
            var configuration = new ConfigurationBuilder()
                .AddJsonFile("appsettings.Integration.json")
                .Build();

            builder.ConfigureTestServices(services =>
            {
                /* setup whatever services you need to override, 
                its useful for overriding db context if you're using Entity Framework */
                var dbConnectionString = configuration.GetSection("ConnectionStrings")["TheConnectionString"];

                services.AddDbContext<DbContext>(optionsBuilder =>
                {
                    optionsBuilder.UseSqlServer(dbConnectionString);
                });

            })
            .CreateClient(new WebApplicationFactoryClientOptions
            {
                AllowAutoRedirect = false
            });
        }
```

The fixture implements IClassFixture<WebApplicationFactory<Startup>>. This lets us:
- Modify Services configuration, for example changing DbContext's connection string (might be useful with Integration testing)
- Overriding some other services, for example Azure KeyVaults, if you're using them for secrets management
- Sets up TestServer (this comes with WebApplicationFactory)
- Create http client

I've also included method that is serializing data to json and adds necessary headers.
It helps me since my integration tests are checking if our REST Api is working as expected.

## Writting tests
This setup is letting us write test in very simple way.
One requirement is that the test class inherits from TestFixture that has been prepared previously.

```csharp
public class Tests : TestFixtureWithAuthentication
    {
        public Tests(WebApplicationFactory<Startup> factory) : base(factory)
        {
        }
    }
```
This will host our API on test server and create the http client.
All we have to do now is http call and assert the response.

```csharp

[Fact]
public async Task Tested_Method_should_return_success_response()
{
    var content = ConvertToHttpContent(new 
    {
        /* Expected post body */
    });

    var response = await Client.PostAsync("/api/Tested_Endpoint", content);

    response.EnsureSuccessStatusCode();
}

```

We can as well deserialize the response and perform more assertions.
I highly recommend FluentAssertions for that. It makes the code elegant and easy to read.

## Adding Authorization Schema to client & server
Authorization on our server side can make testing painful. In past I saw really dirty ways of testing api. Now it's pretty simple.
All you have to do is to extend TestFixture that we've prepared earlier.

At first, we'll introduce TestAuthenticationHandler class. I keep it internal because I don't really want it to be used outside of this project.

```csharp
internal sealed class TestAuthHandler : AuthenticationHandler<AuthenticationSchemeOptions>
    {
        public TestAuthHandler(
            IOptionsMonitor<AuthenticationSchemeOptions> options,
            ILoggerFactory logger,
            UrlEncoder encoder,
            ISystemClock clock) : base(options, logger, encoder, clock)
        {
        }

        protected override Task<AuthenticateResult> HandleAuthenticateAsync()
        {
            var claims = new[]
            {
                new Claim(ClaimTypes.Name, "Test user"),
                new Claim(ClaimTypes.NameIdentifier, Guid.NewGuid().ToString()),
                new Claim("CustomClaimThatYouNeed", "Custom Claim Value"), // Could be some Authorization policy
                new Claim("tenant", "integrationtest") // Useful for multitenancy, if your dbContext depends on that
            };
            
            var identity = new ClaimsIdentity(claims, "Test");
            var principal = new ClaimsPrincipal(identity);
            var ticket = new AuthenticationTicket(principal, "Test");

            var result = AuthenticateResult.Success(ticket);

            return Task.FromResult(result);
        }
    }
```

Extending SetupClient method

```csharp
private void SetupClient()
{
    Client = Factory.WithWebHostBuilder(builder =>
    {
        /* I am telling builder to use appsettings.Integration,
        that is correctly set up in my DevOps pipeline */
        builder.UseEnvironment("Integration");
        var configuration = new ConfigurationBuilder()
            .AddJsonFile("appsettings.Integration.json")
            .Build();

        builder.ConfigureTestServices(services =>
        {
            /* setup whatever services you need to override, 
            its useful for overriding db context if you're using Entity Framework */
            var dbConnectionString = configuration.GetSection("ConnectionStrings")["TheConnectionString"];

            services.AddDbContext<DbContext>(optionsBuilder =>
            {
                optionsBuilder.UseSqlServer(dbConnectionString);
            });

            /* Overriding policies and adding Test Scheme defined in TestAuthHandler */
            services.AddMvc(options =>
            {
                var policy = new AuthorizationPolicyBuilder()
                    .RequireAuthenticatedUser()
                    .AddAuthenticationSchemes("Test")
                    .Build();

                options.Filters.Add(new AuthorizeFilter(policy));
            });

            /* Adding Default Authentication schemes
                This is enough to prevent unauthorized client (if you wish to have that kind of tests) */
            services.AddAuthentication(options =>
            {
                options.DefaultAuthenticateScheme = "Test";
                options.DefaultChallengeScheme = "Test";
                options.DefaultScheme = "Test";
            })
            /* Here we're adding TestAuthentication Scheme with Handler that will Authenticate our client based on Claims */
            .AddScheme<AuthenticationSchemeOptions, TestAuthHandler>("Test", options => { });
        })
        .CreateClient(new WebApplicationFactoryClientOptions
        {
            AllowAutoRedirect = false
        });
    }
}
```

So this extended version is now making a real use of builder.ConfigureTestServices.
It's overriding Mvc and setting it up to use TestPolicy instead.
It's also adding Authentication Scheme and setting defaults for it.
If you have some special stuff in start up, like middlewares and OData or Swagger, ConfigureTestServices is a good place to take care of that.

## Summary
As you can see it's very easy to set up integration testing in .NET Core.
I hope you can make a good use of it in project.
I don't want to write about pros and cons of integration tests here, but it's really important to emphasise that with this configuration it's very easy to treat your api as just a blackbox, where you check response for request. 
Make use of that!

If you have any questions feel free to leave comment!
