2. Modèles de Données
Models/User.cs:

csharp
Copier le code
using System.ComponentModel.DataAnnotations;

namespace AdminInterfaceApp.Models
{
    public class User
    {
        public int Id { get; set; }

        [Required]
        public string Username { get; set; }

        [Required]
        [DataType(DataType.Password)]
        public string Password { get; set; }
    }
}
Models/Photo.cs:

csharp
Copier le code
using System.ComponentModel.DataAnnotations;

namespace AdminInterfaceApp.Models
{
    public class Photo
    {
        public int Id { get; set; }

        [Required]
        public string Url { get; set; }

        [Required]
        public string Description { get; set; }
    }
}
Models/TextContent.cs:

csharp
Copier le code
using System.ComponentModel.DataAnnotations;

namespace AdminInterfaceApp.Models
{
    public class TextContent
    {
        public int Id { get; set; }

        [Required]
        public string Title { get; set; }

        [Required]
        public string Content { get; set; }
    }
}
Models/Logo.cs:

csharp
Copier le code
using System.ComponentModel.DataAnnotations;

namespace AdminInterfaceApp.Models
{
    public class Logo
    {
        public int Id { get; set; }

        [Required]
        public string Url { get; set; }

        [Required]
        public string Description { get; set; }
    }
}
3. Context de la Base de Données
Data/ApplicationDbContext.cs:

csharp
Copier le code
using Microsoft.EntityFrameworkCore;
using AdminInterfaceApp.Models;

namespace AdminInterfaceApp.Data
{
    public class ApplicationDbContext : DbContext
    {
        public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
            : base(options)
        {
        }

        public DbSet<User> Users { get; set; }
        public DbSet<Photo> Photos { get; set; }
        public DbSet<TextContent> TextContents { get; set; }
        public DbSet<Logo> Logos { get; set; }
    }
}
Configurez le contexte de la base de données dans Startup.cs:

csharp
Copier le code
public void ConfigureServices(IServiceCollection services)
{
    services.AddDbContext<ApplicationDbContext>(options =>
        options.UseSqlServer(Configuration.GetConnectionString("DefaultConnection")));
    services.AddControllersWithViews();
}
Ajoutez la chaîne de connexion dans appsettings.json:

json
Copier le code
"ConnectionStrings": {
    "DefaultConnection": "Server=(localdb)\\mssqllocaldb;Database=AdminInterfaceDb;Trusted_Connection=True;MultipleActiveResultSets=true"
}
4. Contrôleurs
Controllers/AdminController.cs:

csharp
Copier le code
using Microsoft.AspNetCore.Mvc;
using AdminInterfaceApp.Data;
using AdminInterfaceApp.Models;
using System.Threading.Tasks;
using Microsoft.EntityFrameworkCore;

namespace AdminInterfaceApp.Controllers
{
    public class AdminController : Controller
    {
        private readonly ApplicationDbContext _context;

        public AdminController(ApplicationDbContext context)
        {
            _context = context;
        }

        // Photos
        public async Task<IActionResult> Photos()
        {
            return View(await _context.Photos.ToListAsync());
        }

        public IActionResult CreatePhoto()
        {
            return View();
        }

        [HttpPost]
        [ValidateAntiForgeryToken]
        public async Task<IActionResult> CreatePhoto(Photo photo)
        {
            if (ModelState.IsValid)
            {
                _context.Add(photo);
                await _context.SaveChangesAsync();
                return RedirectToAction(nameof(Photos));
            }
            return View(photo);
        }

        public async Task<IActionResult> EditPhoto(int? id)
        {
            if (id == null)
            {
                return NotFound();
            }

            var photo = await _context.Photos.FindAsync(id);
            if (photo == null)
            {
                return NotFound();
            }
            return View(photo);
        }

        [HttpPost]
        [ValidateAntiForgeryToken]
        public async Task<IActionResult> EditPhoto(int id, Photo photo)
        {
            if (id != photo.Id)
            {
                return NotFound();
            }

            if (ModelState.IsValid)
            {
                try
                {
                    _context.Update(photo);
                    await _context.SaveChangesAsync();
                }
                catch (DbUpdateConcurrencyException)
                {
                    if (!PhotoExists(photo.Id))
                    {
                        return NotFound();
                    }
                    else
                    {
                        throw;
                    }
                }
                return RedirectToAction(nameof(Photos));
            }
            return View(photo);
        }

        public async Task<IActionResult> DeletePhoto(int? id)
        {
            if (id == null)
            {
                return NotFound();
            }

            var photo = await _context.Photos
                .FirstOrDefaultAsync(m => m.Id == id);
            if (photo == null)
            {
                return NotFound();
            }

            return View(photo);
        }

        [HttpPost, ActionName("DeletePhoto")]
        [ValidateAntiForgeryToken]
        public async Task<IActionResult> DeleteConfirmedPhoto(int id)
        {
            var photo = await _context.Photos.FindAsync(id);
            _context.Photos.Remove(photo);
            await _context.SaveChangesAsync();
            return RedirectToAction(nameof(Photos));
        }

        private bool PhotoExists(int id)
        {
            return _context.Photos.Any(e => e.Id == id);
        }

        // TextContent
        public async Task<IActionResult> TextContents()
        {
            return View(await _context.TextContents.ToListAsync());
        }

        public IActionResult CreateTextContent()
        {
            return View();
        }

        [HttpPost]
        [ValidateAntiForgeryToken]
        public async Task<IActionResult> CreateTextContent(TextContent textContent)
        {
            if (ModelState.IsValid)
            {
                _context.Add(textContent);
                await _context.SaveChangesAsync();
                return RedirectToAction(nameof(TextContents));
            }
            return View(textContent);
        }

        public async Task<IActionResult> EditTextContent(int? id)
        {
            if (id == null)
            {
                return NotFound();
            }

            var textContent = await _context.TextContents.FindAsync(id);
            if (textContent == null)
            {
                return NotFound();
            }
            return View(textContent);
        }

        [HttpPost]
        [ValidateAntiForgeryToken]
        public async Task<IActionResult> EditTextContent(int id, TextContent textContent)
        {
            if (id != textContent.Id)
            {
                return NotFound();
            }

            if (ModelState.IsValid)
            {
                try
                {
                    _context.Update(textContent);
                    await _context.SaveChangesAsync();
                }
                catch (DbUpdateConcurrencyException)
                {
                    if (!TextContentExists(textContent.Id))
                    {
                        return NotFound();
                    }
                    else
                    {
                        throw;
                    }
                }
                return RedirectToAction(nameof(TextContents));
            }
            return View(textContent);
        }

        public async Task<IActionResult> DeleteTextContent(int? id)
        {
            if (id == null)
            {
                return NotFound();
            }

            var textContent = await _context.TextContents
                .FirstOrDefaultAsync(m => m.Id == id);
            if (textContent == null)
            {
                return NotFound();
            }

            return View(textContent);
        }

        [HttpPost, ActionName("DeleteTextContent")]
        [ValidateAntiForgeryToken]
        public async Task<IActionResult> DeleteConfirmedTextContent(int id)
        {
            var textContent = await _context.TextContents.FindAsync(id);
            _context.TextContents.Remove(textContent);
            await _context.SaveChangesAsync();
            return RedirectToAction(nameof(TextContents));
        }

        private bool TextContentExists(int id)
        {
            return _context.TextContents.Any(e => e.Id == id);
        }

        // Logos
        public async Task<IActionResult> Logos()
        {
            return View(await _context.Logos.ToListAsync());
        }

        public IActionResult CreateLogo()
        {
            return View();
        }

        [HttpPost]
        [ValidateAntiForgeryToken]
        public async Task<IActionResult> CreateLogo(Logo logo)
        {
            if (ModelState.IsValid)
            {
                _context.Add(logo);
                await _context.SaveChangesAsync();
                return RedirectToAction(nameof(Logos));
            }
            return View(logo);
        }

        public async Task<IActionResult> EditLogo(int? id)
        {
            if (id == null)
            {
                return NotFound();
            }

            var logo = await _context.Logos.FindAsync(id);
            if (logo == null)
            {
                return NotFound();
            }
            return View(logo);
        }

        [HttpPost]
        [ValidateAntiForgeryToken]
        public async Task<IActionResult> EditLogo(int id, Logo logo)
        {
            if (id != logo.Id)
            {
                return NotFound();
            }

            if (ModelState.IsValid)
            {
                try
                {
                    _context.Update(logo);
                    await _context.SaveChangesAsync();
                }
                catch (DbUpdateConcurrencyException)
                {
                    if (!LogoExists(logo.Id))
                    {
                        return NotFound();
                    }
                    else
                    {
                        throw;
                    }
                }
                return RedirectToAction(nameof(Logos));
            }
            return View(logo);
        }

        public async Task<IActionResult> DeleteLogo(int? id)
        {
            if (id == null)
            {
                return NotFound();
            }

            var logo = await _context.Logos
                .FirstOrDefaultAsync(m => m.Id == id);
            if (logo == null)
            {
                return NotFound();
            }

            return View(logo);
        }

        [HttpPost, ActionName("DeleteLogo")]
        [ValidateAntiForgeryToken]
        public async Task<IActionResult> DeleteConfirmedLogo(int id)
        {
            var logo = await _context.Logos.FindAsync(id);
            _context.Logos.Remove(logo);
            await _context.SaveChangesAsync();
            return RedirectToAction(nameof(Logos));
        }

        private bool LogoExists(int id)
        {
            return _context.Logos.Any(e => e.Id == id);
        }

        // Users
        public async Task<IActionResult> Users()
        {
            return View(await _context.Users.ToListAsync());
        }

        public IActionResult CreateUser()
        {
            return View();
        }

        [HttpPost]
        [ValidateAntiForgeryToken]
        public async Task<IActionResult> CreateUser(User user)
        {
            if (ModelState.IsValid)
            {
                _context.Add(user);
                await _context.SaveChangesAsync();
                return RedirectToAction(nameof(Users));
            }
            return View(user);
        }

        public async Task<IActionResult> EditUser(int? id)
        {
            if (id == null)
            {
                return NotFound();
            }

            var user = await _context.Users.FindAsync(id);
            if (user == null)
            {
                return NotFound();
            }
            return View(user);
        }

        [HttpPost]
        [ValidateAntiForgeryToken]
        public async Task<IActionResult> EditUser(int id, User user)
        {
            if (id != user.Id)
            {
                return NotFound();
            }

            if (ModelState.IsValid)
            {
                try
                {
                    _context.Update(user);
                    await _context.SaveChangesAsync();
                }
                catch (DbUpdateConcurrencyException)
                {
                    if (!UserExists(user.Id))
                    {
                        return NotFound();
                    }
                    else
                    {
                        throw;
                    }
                }
                return RedirectToAction(nameof(Users));
            }
            return View(user);
        }

        public async Task<IActionResult> DeleteUser(int? id)
        {
            if (id == null)
            {
                return NotFound();
            }

            var user = await _context.Users
                .FirstOrDefaultAsync(m => m.Id == id);
            if (user == null)
            {
                return NotFound();
            }

            return View(user);
        }

        [HttpPost, ActionName("DeleteUser")]
        [ValidateAntiForgeryToken]
        public async Task<IActionResult> DeleteConfirmedUser(int id)
        {
            var user = await _context.Users.FindAsync(id);
            _context.Users.Remove(user);
            await _context.SaveChangesAsync();
            return RedirectToAction(nameof(Users));
        }

        private bool UserExists(int id)
        {
            return _context.Users.Any(e => e.Id == id);
        }
    }
}
5. Vues
Créez des vues Razor pour chaque action dans le contrôleur. Voici un exemple pour l'index des photos.

Views/Admin/Photos.cshtml:

html
Copier le code
@model IEnumerable<AdminInterfaceApp.Models.Photo>

@{
    ViewData["Title"] = "Photos";
}

<h1>Photos</h1>

<p>
    <a asp-action="CreatePhoto">Add New Photo</a>
</p>

<table class="table">
    <thead>
        <tr>
            <th>
                @Html.DisplayNameFor(model => model.Description)
            </th>
            <th>
                @Html.DisplayNameFor(model => model.Url)
            </th>
            <th></th>
        </tr>
    </thead>
    <tbody>
@foreach (var item in Model) {
        <tr>
            <td>
                @Html.DisplayFor(modelItem => item.Description)
            </td>
            <td>
                <img src="@item.Url" alt="Photo" width="100" height="100"/>
            </td>
            <td>
                <a asp-action="EditPhoto" asp-route-id="@item.Id">Edit</a> |
                <a asp-action="DeletePhoto" asp-route-id="@item.Id">Delete</a>
            </td>
        </tr>
}
    </tbody>
</table>
6. Authentification et Autorisation
Ajoutez la prise en charge de l'authentification dans Startup.cs:

csharp
Copier le code
public void ConfigureServices(IServiceCollection services)
{
    services.AddDbContext<ApplicationDbContext>(options =>
        options.UseSqlServer(Configuration.GetConnectionString("DefaultConnection")));
    services.AddControllersWithViews();

    services.AddIdentity<User, IdentityRole>()
        .AddEntityFrameworkStores<ApplicationDbContext>()
        .AddDefaultTokenProviders();

    services.Configure<IdentityOptions>(options =>
    {
        options.Password.RequireDigit = true;
        options.Password.RequireLowercase = true;
        options.Password.RequireNonAlphanumeric = false;
        options.Password.RequireUppercase = true;
        options.Password.RequiredLength = 6;
    });
}

public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    if (env.IsDevelopment())
    {
        app.UseDeveloperExceptionPage();
    }
    else
    {
        app.UseExceptionHandler("/Home/Error");
        app.UseHsts();
    }
    app.UseHttpsRedirection();
    app.UseStaticFiles();

    app.UseRouting();

    app.UseAuthentication();
    app.UseAuthorization();

    app.UseEndpoints(endpoints =>
    {
        endpoints.MapControllerRoute(
            name: "default",
            pattern: "{controller=Home}/{action=Index}/{id?}");
    });
}
Conclusion
Ce guide offre une base pour créer une application d'administration en ASP.NET Core, permettant de gérer les photos, les textes, les utilisateurs, les logos, et de voir les utilisateurs connectés, avec une connexion sécurisée. Pour un projet réel, vous devrez ajouter des fonctionnalités supplémentaires et des validations, ainsi qu'une gestion plus avancée des utilisateurs et des rôles.




4o
peux tu des fonctionnalités supplementaires et des validations de gestion avancée

Pour ajouter des fonctionnalités supplémentaires et des validations de gestion avancée à notre interface d'administration en ASP.NET, nous pouvons suivre ces étapes :

Ajout de validations sur les modèles.
Gestion des rôles et permissions utilisateurs.
Ajout de fonctionnalités de pagination et de recherche.
Amélioration des vues avec Bootstrap pour une meilleure interface utilisateur.
1. Ajout de validations sur les modèles
Nous pouvons ajouter des validations de données à nos modèles en utilisant les attributs de validation intégrés.

Models/User.cs:

csharp
Copier le code
using System.ComponentModel.DataAnnotations;

namespace AdminInterfaceApp.Models
{
    public class User
    {
        public int Id { get; set; }

        [Required(ErrorMessage = "Le nom d'utilisateur est requis")]
        [StringLength(100, ErrorMessage = "Le nom d'utilisateur doit comporter au moins {2} et au maximum {1} caractères.", MinimumLength = 6)]
        public string Username { get; set; }

        [Required(ErrorMessage = "Le mot de passe est requis")]
        [DataType(DataType.Password)]
        [StringLength(100, ErrorMessage = "Le mot de passe doit comporter au moins {2} et au maximum {1} caractères.", MinimumLength = 6)]
        public string Password { get; set; }
    }
}
Models/Photo.cs:

csharp
Copier le code
using System.ComponentModel.DataAnnotations;

namespace AdminInterfaceApp.Models
{
    public class Photo
    {
        public int Id { get; set; }

        [Required(ErrorMessage = "L'URL de la photo est requise")]
        [Url(ErrorMessage = "L'URL de la photo n'est pas valide")]
        public string Url { get; set; }

        [Required(ErrorMessage = "La description de la photo est requise")]
        [StringLength(500, ErrorMessage = "La description doit comporter au moins {2} et au maximum {1} caractères.", MinimumLength = 10)]
        public string Description { get; set; }
    }
}
2. Gestion des rôles et permissions utilisateurs
Nous allons créer des rôles et attribuer des permissions spécifiques aux utilisateurs. Pour cela, nous devons configurer Identity dans Startup.cs et ajouter une logique pour créer des rôles.

Startup.cs:

csharp
Copier le code
public void ConfigureServices(IServiceCollection services)
{
    services.AddDbContext<ApplicationDbContext>(options =>
        options.UseSqlServer(Configuration.GetConnectionString("DefaultConnection")));
    services.AddControllersWithViews();

    services.AddIdentity<User, IdentityRole>()
        .AddEntityFrameworkStores<ApplicationDbContext>()
        .AddDefaultTokenProviders();

    services.Configure<IdentityOptions>(options =>
    {
        options.Password.RequireDigit = true;
        options.Password.RequireLowercase = true;
        options.Password.RequireNonAlphanumeric = false;
        options.Password.RequireUppercase = true;
        options.Password.RequiredLength = 6;
    });

    services.AddAuthorization(options =>
    {
        options.AddPolicy("RequireAdminRole", policy => policy.RequireRole("Admin"));
        options.AddPolicy("RequireUserRole", policy => policy.RequireRole("User"));
    });
}

public void Configure(IApplicationBuilder app, IWebHostEnvironment env, IServiceProvider serviceProvider)
{
    if (env.IsDevelopment())
    {
        app.UseDeveloperExceptionPage();
    }
    else
    {
        app.UseExceptionHandler("/Home/Error");
        app.UseHsts();
    }
    app.UseHttpsRedirection();
    app.UseStaticFiles();

    app.UseRouting();

    app.UseAuthentication();
    app.UseAuthorization();

    app.UseEndpoints(endpoints =>
    {
        endpoints.MapControllerRoute(
            name: "default",
            pattern: "{controller=Home}/{action=Index}/{id?}");
    });

    CreateRoles(serviceProvider).Wait();
}

private async Task CreateRoles(IServiceProvider serviceProvider)
{
    var RoleManager = serviceProvider.GetRequiredService<RoleManager<IdentityRole>>();
    var UserManager = serviceProvider.GetRequiredService<UserManager<User>>();
    string[] roleNames = { "Admin", "User" };
    IdentityResult roleResult;

    foreach (var roleName in roleNames)
    {
        var roleExist = await RoleManager.RoleExistsAsync(roleName);
        if (!roleExist)
        {
            roleResult = await RoleManager.CreateAsync(new IdentityRole(roleName));
        }
    }

    User user = await UserManager.FindByEmailAsync("admin@admin.com");

    if (user == null)
    {
        user = new User()
        {
            UserName = "admin@admin.com",
            Email = "admin@admin.com",
        };
        await UserManager.CreateAsync(user, "Admin@123456");
    }
    await UserManager.AddToRoleAsync(user, "Admin");
}
3. Ajout de fonctionnalités de pagination et de recherche
Pour la pagination, nous pouvons utiliser une bibliothèque comme X.PagedList.

Ajoutez X.PagedList à votre projet via NuGet :

sh
Copier le code
dotnet add package X.PagedList.Mvc.Core
Controllers/AdminController.cs:

csharp
Copier le code
using X.PagedList;

public async Task<IActionResult> Photos(int? page)
{
    var pageNumber = page ?? 1;
    var photos = await _context.Photos.ToPagedListAsync(pageNumber, 10);
    return View(photos);
}
Views/Admin/Photos.cshtml:

html
Copier le code
@model IPagedList<AdminInterfaceApp.Models.Photo>

@{
    ViewData["Title"] = "Photos";
}

<h1>Photos</h1>

<p>
    <a asp-action="CreatePhoto">Add New Photo</a>
</p>

<table class="table">
    <thead>
        <tr>
            <th>@Html.DisplayNameFor(model => model.Description)</th>
            <th>@Html.DisplayNameFor(model => model.Url)</th>
            <th></th>
        </tr>
    </thead>
    <tbody>
@foreach (var item in Model) {
        <tr>
            <td>@Html.DisplayFor(modelItem => item.Description)</td>
            <td><img src="@item.Url" alt="Photo" width="100" height="100"/></td>
            <td>
                <a asp-action="EditPhoto" asp-route-id="@item.Id">Edit</a> |
                <a asp-action="DeletePhoto" asp-route-id="@item.Id">Delete</a>
            </td>
        </tr>
}
    </tbody>
</table>

<div class="pager">
    @Html.PagedListPager(Model, page => Url.Action("Photos", new { page }))
</div>
Pour la recherche, nous pouvons ajouter un formulaire de recherche et une méthode pour filtrer les résultats.

Controllers/AdminController.cs:

csharp
Copier le code
public async Task<IActionResult> Photos(string searchString, int? page)
{
    var pageNumber = page ?? 1;
    var photos = from p in _context.Photos select p;

    if (!String.IsNullOrEmpty(searchString))
    {
        photos = photos.Where(s => s.Description.Contains(searchString));
    }

    var pagedPhotos = await photos.ToPagedListAsync(pageNumber, 10);
    return View(pagedPhotos);
}
Views/Admin/Photos.cshtml:

html
Copier le code
@model IPagedList<AdminInterfaceApp.Models.Photo>

@{
    ViewData["Title"] = "Photos";
}

<h1>Photos</h1>

<form asp-action="Photos">
    <div class="form-group">
        <input type="text" class="form-control" name="searchString" value="@ViewData["CurrentFilter"]" placeholder="Search by description">
    </div>
    <button type="submit" class="btn btn-primary">Search</button>
</form>

<p>
    <a asp-action="CreatePhoto">Add New Photo</a>
</p>

<table class="table">
    <thead>
        <tr>
            <th>@Html.DisplayNameFor(model => model.Description)</th>
            <th>@Html.DisplayNameFor(model => model.Url)</th>
            <th></th>
        </tr>
    </thead>
    <tbody>
@foreach (var item in Model) {
        <tr>
            <td>@Html.DisplayFor(modelItem => item.Description)</td>
            <td><img src="@item.Url" alt="Photo" width="100" height="100"/></td>
            <td>
                <a asp-action="EditPhoto" asp-route-id="@item.Id">Edit</a> |
                <a asp-action="DeletePhoto" asp-route-id="@item.Id">Delete</a>
            </td>
        </tr>
}
    </tbody>
</table>

<div class="pager">
    @Html.PagedListPager(Model, page => Url.Action("Photos", new { page }))
</div>
4. Amélioration des vues avec Bootstrap
Pour une interface utilisateur plus agréable, utilisez Bootstrap. Ajoutez Bootstrap à votre projet :

_Layout.cshtml:

html
Copier le code
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>@ViewData["Title"] - AdminInterfaceApp</title>
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/css/bootstrap.min.css" />
</head>
<body>
    <div class="container">
        <main role="main" class="pb-3">
            @RenderBody()
        </main>
    </div>

    <script src="https://code.jquery.com/jquery-3.5.1.slim.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@4.5.2/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
Ensuite, utilisez les classes Bootstrap dans vos vues pour améliorer la mise en page et le style.

Views/Admin/Photos.cshtml:

html
Copier le code
@model IPagedList<AdminInterfaceApp.Models.Photo>

@{
    ViewData["Title"] = "Photos";
}

<h1 class="my-4">Photos</h1>

<form asp-action="Photos" class="form-inline my-2 my-lg-0">
    <input class="form-control mr-sm-2" type="search" name="searchString" value="@ViewData["CurrentFilter"]" placeholder="Search by description">
    <button class="btn btn-outline-success my-2 my-sm-0" type="submit">Search</button>
</form>

<p>
    <a class="btn btn-primary my-3" asp-action="CreatePhoto">Add New Photo</a>
</p>

<table class="table table-striped">
    <thead class="thead-dark">
        <tr>
            <th>@Html.DisplayNameFor(model => model.Description)</th>
            <th>@Html.DisplayNameFor(model => model.Url)</th>
            <th></th>
        </tr>
    </thead>
    <tbody>
@foreach (var item in Model) {
        <tr>
            <td>@Html.DisplayFor(modelItem => item.Description)</td>
            <td><img src="@item.Url" alt="Photo" width="100" height="100"/></td>
            <td>
                <a class="btn btn-sm btn-warning" asp-action="EditPhoto" asp-route-id="@item.Id">Edit</a>
                <a class="btn btn-sm btn-danger" asp-action="DeletePhoto" asp-route-id="@item.Id">Delete</a>
            </td>
        </tr>
}
    </tbody>
</table>

<div class="pager">
    @Html.PagedListPager(Model, page => Url.Action("Photos", new { page }), new PagedListRenderOptions { Bootstrap4 = true })
</div>
