# Identity med roller
Ett litet enkelt exempel med .NET identity där roller används.

Längst ner i Program.cs skapas tre roller med "seeding"; administrator, manager och user.

Sedan skapas två användare, som tilldelas olika roller.

Slutligen har åtkomst skyddats till Privacy-sidan, som enbart ges åtkomst för användare med rollen "Administrator".

## Steg för steg
1: Skapa ett nytt MVC-projekt med Identity:
```bash
dotnet new mvc --auth Individual -o IdentityWithRoles
```
2: Lägg till raden `.AddRoles<IdentityRole>()` till raden för AddDefaultIdentity:
```bash
builder.Services.AddDefaultIdentity<IdentityUser>(options => options.SignIn.RequireConfirmedAccount = false)
    .AddRoles<IdentityRole>()
    .AddEntityFrameworkStores<ApplicationDbContext>();
```
3: Lägg till kod för att skapa roller med **RoleManager** och användare med **UserManager**, förslagvis innan raden `app.Run();` längst ner i **Program.cs**:
```bash
// Seeder
using (var scope = app.Services.CreateScope())
{
    var roleManager = scope.ServiceProvider.GetRequiredService<RoleManager<IdentityRole>>();

    // Create roles if they don't exist
    var roles = new[] { "Administrator", "Manager", "User" };
    foreach (var role in roles)
    {
        if (!await roleManager.RoleExistsAsync(role))
        {
            await roleManager.CreateAsync(new IdentityRole(role));
        }
    }

    // Add new users and assign role
    var userManager = scope.ServiceProvider.GetRequiredService<UserManager<IdentityUser>>();

    // Credentials
    var users = new[] {
        new { Email = "mattias@miun.se", Password = "Password123!", Role = "Administrator" },
        new { Email = "user@miun.se", Password = "Password123!", Role = "User" }
    };

    foreach (var user in users)
    {
        var identityUser = await userManager.FindByEmailAsync(user.Email);
        if (identityUser == null)
        {
            identityUser = new IdentityUser { UserName = user.Email, Email = user.Email };
            await userManager.CreateAsync(identityUser, user.Password);

            // Assign role
            await userManager.AddToRoleAsync(identityUser, user.Role);
        }
    }
}
```

4: Skydda routes i exempelvis Controllern **HomeController.cs** med:
```bash
[Authorize(Roles = "Administrator")]
public IActionResult Privacy()
{
    return View();
}
```

5: Starta applikation med `dotnet run`, logga in med någon av användarna från punkt 3, och testa att nå Privacy-sidan.

## Av
Mattias Dahlgren, mattias.dahlgren@miun.se