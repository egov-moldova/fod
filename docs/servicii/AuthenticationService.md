# AuthenticationService

## Descriere Generală

`AuthenticationService` este serviciul responsabil pentru obținerea informațiilor despre utilizatorul curent autentificat. Serviciul comunică cu endpoint-ul de autentificare pentru a prelua claims-urile utilizatorului și a determina starea de autentificare.

## Configurare

### Înregistrare în Dependency Injection

```csharp
// În Program.cs sau Startup.cs
builder.Services.AddHttpClient<IAuthenticationService, AuthenticationService>(client =>
{
    client.BaseAddress = new Uri("https://api.example.com/");
});

// Sau cu configurare specifică
builder.Services.AddScoped<IAuthenticationService>(sp =>
{
    var httpClient = sp.GetRequiredService<HttpClient>();
    return new AuthenticationService(httpClient);
});
```

### Configurare HttpClient

```csharp
builder.Services.AddHttpClient<IAuthenticationService, AuthenticationService>(client =>
{
    client.BaseAddress = new Uri(Configuration["ApiBaseUrl"]);
    client.DefaultRequestHeaders.Add("Accept", "application/json");
})
.AddHttpMessageHandler<AuthenticationHandler>(); // Pentru token management
```

## Interfața IAuthenticationService

```csharp
public interface IAuthenticationService
{
    Task<CurrentUser> CurrentUserInfo();
}
```

## Metode disponibile

### CurrentUserInfo()

Obține informațiile despre utilizatorul curent autentificat.

**Parametri:** Niciun parametru

**Returnează:** `Task<CurrentUser>` - Informațiile utilizatorului curent

**Comportament:**
- Face request GET către `/account/me`
- Dacă răspunsul este NoContent (204), returnează un CurrentUser gol (neautentificat)
- Parsează răspunsul JSON și extrage toate claim-urile
- Gestionează claim-uri cu valori multiple (array)

## Exemple de utilizare

### Verificare autentificare în componentă

```razor
@inject IAuthenticationService AuthService

@if (currentUser?.IsAuthenticated == true)
{
    <p>Bine ai venit, @GetUserName()!</p>
}
else
{
    <p>Nu ești autentificat.</p>
}

@code {
    private CurrentUser currentUser;

    protected override async Task OnInitializedAsync()
    {
        currentUser = await AuthService.CurrentUserInfo();
    }

    private string GetUserName()
    {
        return currentUser?.Claims?
            .FirstOrDefault(c => c.Key == "name")?.Value 
            ?? "Utilizator";
    }
}
```

### Obținere roluri utilizator

```razor
@inject IAuthenticationService AuthService

@code {
    private List<string> userRoles = new();

    protected override async Task OnInitializedAsync()
    {
        var user = await AuthService.CurrentUserInfo();
        if (user.IsAuthenticated)
        {
            userRoles = user.Claims
                .Where(c => c.Key == "role")
                .Select(c => c.Value)
                .ToList();
        }
    }

    private bool HasRole(string role)
    {
        return userRoles.Contains(role);
    }
}
```

### AuthenticationStateProvider custom

```csharp
public class CustomAuthenticationStateProvider : AuthenticationStateProvider
{
    private readonly IAuthenticationService _authService;

    public CustomAuthenticationStateProvider(IAuthenticationService authService)
    {
        _authService = authService;
    }

    public override async Task<AuthenticationState> GetAuthenticationStateAsync()
    {
        var currentUser = await _authService.CurrentUserInfo();
        
        if (!currentUser.IsAuthenticated)
        {
            return new AuthenticationState(new ClaimsPrincipal(new ClaimsIdentity()));
        }

        var claims = currentUser.Claims
            .Select(c => new Claim(c.Key, c.Value))
            .ToList();

        var identity = new ClaimsIdentity(claims, "apiauth");
        var user = new ClaimsPrincipal(identity);

        return new AuthenticationState(user);
    }
}
```

## Integrare cu alte servicii

### Cu IUserService

```csharp
public class UserService : IUserService
{
    private readonly IAuthenticationService _authService;
    
    public UserService(IAuthenticationService authService)
    {
        _authService = authService;
    }

    public async Task<UserProfile> GetCurrentUserProfile()
    {
        var currentUser = await _authService.CurrentUserInfo();
        if (!currentUser.IsAuthenticated)
            return null;

        var userId = currentUser.Claims
            .FirstOrDefault(c => c.Key == "sub")?.Value;
            
        // Încarcă profilul complet folosind userId
        return await LoadUserProfile(userId);
    }
}
```

### Cu servicii de autorizare

```csharp
public class AuthorizationService
{
    private readonly IAuthenticationService _authService;
    
    public async Task<bool> CanAccessResource(string resource)
    {
        var user = await _authService.CurrentUserInfo();
        if (!user.IsAuthenticated) return false;

        var permissions = user.Claims
            .Where(c => c.Key == "permission")
            .Select(c => c.Value);

        return permissions.Contains(resource);
    }
}
```

## Tratare erori

### Gestionare excepții

```csharp
public class SafeAuthenticationService : IAuthenticationService
{
    private readonly AuthenticationService _innerService;
    private readonly ILogger<SafeAuthenticationService> _logger;

    public async Task<CurrentUser> CurrentUserInfo()
    {
        try
        {
            return await _innerService.CurrentUserInfo();
        }
        catch (HttpRequestException ex)
        {
            _logger.LogError(ex, "Eroare la obținerea informațiilor utilizatorului");
            return new CurrentUser { IsAuthenticated = false };
        }
        catch (TaskCanceledException)
        {
            _logger.LogWarning("Timeout la obținerea informațiilor utilizatorului");
            return new CurrentUser { IsAuthenticated = false };
        }
    }
}
```

### Cu retry policy

```csharp
builder.Services.AddHttpClient<IAuthenticationService, AuthenticationService>()
    .AddPolicyHandler(HttpPolicyExtensions
        .HandleTransientHttpError()
        .WaitAndRetryAsync(
            3,
            retryAttempt => TimeSpan.FromSeconds(Math.Pow(2, retryAttempt)),
            onRetry: (outcome, timespan, retryCount, context) =>
            {
                logger.LogWarning($"Retry {retryCount} după {timespan} secunde");
            }));
```

## Model CurrentUser

```csharp
public class CurrentUser
{
    public bool IsAuthenticated { get; set; }
    public List<KeyValuePair<string, string>> Claims { get; set; } = new();
}
```

### Claims comune

| Claim | Descriere |
|-------|-----------|
| sub | Subject - ID unic utilizator |
| name | Numele complet |
| given_name | Prenume |
| family_name | Nume familie |
| email | Adresă email |
| role | Roluri utilizator (poate fi multiplu) |
| permission | Permisiuni specifice |
| exp | Expirare token |

## Note tehnice

1. **Endpoint fix** - Folosește întotdeauna `/account/me`
2. **Gestionare arrays** - Suportă claims cu valori multiple
3. **No authentication** - Returnează user neautentificat pentru 204
4. **JSON parsing** - Folosește System.Text.Json
5. **Dynamic handling** - Gestionează orice structură de claims

## Bune practici

1. **Caching** - Cache rezultatul pentru a evita request-uri repetate
2. **Error handling** - Gestionați întotdeauna erorile de rețea
3. **Token refresh** - Integrați cu token refresh logic
4. **Null checks** - Verificați întotdeauna IsAuthenticated
5. **Claim validation** - Validați existența claim-urilor înainte de utilizare

## Concluzie

AuthenticationService oferă o interfață simplă pentru obținerea informațiilor despre utilizatorul autentificat. Cu suport pentru claims multiple și gestionare flexibilă a răspunsurilor, serviciul se integrează perfect în aplicațiile Blazor care necesită autentificare.