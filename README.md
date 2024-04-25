# AuthorizeRoleView

AuthorizeRoleView is a .NET 8 Blazor component that authorizes the user based on the roles provided in the Roles parameter. 

It is designed to be a replacement for `AuthorizeView` when used in a Blazor WebAssembly app.

## Requirements:

AuthorizeRoleView  is designed to be used in a Blazor Web App that uses the default authentication and authorization setup (individual accounts) with the interactive render mode set to WebAssembly and the  interactivity location set to Global.

It works whether or not you are using pre-rendering.

It has not been tested with the per-page/component interactivity location.

The server and client must have an `httpClient` configured with the base address of the server. 

On the server side, it looks like this:

```c#
builder.Services.AddScoped(sp => new HttpClient { BaseAddress = new Uri("https://localhost:7217/") });
```

Replace the URL with the base address of your server.

On the client side, it looks like this:

```c#
builder.Services.AddScoped(sp => new HttpClient
{
    BaseAddress = new Uri(builder.HostEnvironment.BaseAddress)
});
```

On the server, you must add roles to your configuration like so:

```c#
builder.Services.AddIdentityCore<ApplicationUser>(options => options.SignIn.RequireConfirmedAccount = true)
    .AddRoles<IdentityRole>()
    .AddEntityFrameworkStores<ApplicationDbContext>()
    .AddSignInManager()
    .AddDefaultTokenProviders();
```

Lastly, there must be an endpoint on the server that returns the roles for the user like so:

```c#
app.MapGet("/api/roles", async (HttpContext httpContext) =>
{
    var user = httpContext.User;
    if (user.Identity?.IsAuthenticated == true)
    {
        // Get roles from the claims
        var roles = user.Claims
                        .Where(c => c.Type == ClaimTypes.Role)
                        .Select(c => c.Value)
                        .ToList();

        if (roles.Any())
        {
            return Results.Ok(roles);
        }
        return Results.NotFound("No roles found for this user.");
    }
    return Results.Unauthorized();
}).RequireAuthorization();
```

## Usage:

You can use the AuthorizeRoleView wherever you would use AuthorizeView.
For example, to limit the visibility of a NavMenu item in the NavMenu.razor file:

```xml
<AuthorizeRoleView Roles="admin, counterClicker">
    <Authorized>
        <div class="nav-item px-3">
            <NavLink class="nav-link" href="counter">
                <span class="bi bi-plus-square-fill-nav-menu" aria-hidden="true"></span> Counter
            </NavLink>
        </div>
    </Authorized>
</AuthorizeRoleView>
```

Roles are specified as a comma-separated string.

Just Like `AuthorizeView`, you do not have to specify an `<Authorized>` tag if all you want to show is the authorized content.

However, you can use `<Authorized>`, `<NotAuthorized>`, and `<Authoriaing>` just as you would with `<AuthorizedView>`.

You can also use the `IsInRole()` method in code, for example in a .razor file:

```c#
<AuthorizeRoleView @ref="authorizeRoleView"/>

@code {
    AuthorizeRoleView authorizeRoleView;

    protected override async Task OnInitializedAsync()
    {
        var isInRole = await authorizeRoleView.IsInRole("admin");
    }
}
```