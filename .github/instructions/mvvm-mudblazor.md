# Implementing MVVM Pattern with MudBlazor in Blazor

## Introduction

This guide provides a comprehensive walkthrough of implementing the Model-View-ViewModel (MVVM) architectural pattern with MudBlazor in a Blazor application. By the end of this guide, you'll understand how to create maintainable, testable, and visually appealing Blazor applications using these powerful tools.

## 1. Overview of MVVM and MudBlazor

### What is MVVM?

The Model-View-ViewModel (MVVM) pattern is a software architectural pattern that facilitates the separation of the development of the graphical user interface from the development of business logic or back-end logic. It consists of three main components:

- **Model**: Represents the data and business logic of the application. It contains the core entities and domain logic, independent of the user interface.

- **View**: The user interface layer that displays data to the user. In Blazor, this is typically a Razor component (.razor file).

- **ViewModel**: Acts as an intermediary between the View and the Model. It exposes data and commands that the View can bind to, handles user interactions, and encapsulates presentation logic.

**Benefits of MVVM:**
- **Separation of Concerns**: Clear separation between UI, business logic, and data layers
- **Testability**: ViewModels can be unit tested without the UI
- **Maintainability**: Changes to the UI don't affect business logic and vice versa
- **Reusability**: ViewModels can be reused across different Views
- **Collaboration**: Designers can work on Views while developers focus on ViewModels

### What is MudBlazor?

MudBlazor is an ambitious Material Design component library for Blazor. It provides a comprehensive set of pre-built UI components that follow Google's Material Design principles, offering:

- **Rich Component Library**: Buttons, forms, tables, dialogs, navigation, and more
- **Customizable Themes**: Easy theming system to match your brand
- **Responsive Design**: Mobile-first, responsive components
- **Accessibility**: Built with accessibility in mind
- **Performance**: Optimized for Blazor Server and WebAssembly
- **Active Community**: Well-documented with regular updates

MudBlazor makes it easy to create professional, modern web applications without having to build UI components from scratch.

## 2. Project Setup

### Creating a New Blazor Project

1. **Create a new Blazor Server App:**
   ```bash
   dotnet new blazorserver -n MudBlazorMvvmDemo
   cd MudBlazorMvvmDemo
   ```

   Or for Blazor WebAssembly:
   ```bash
   dotnet new blazorwasm -n MudBlazorMvvmDemo
   cd MudBlazorMvvmDemo
   ```

2. **Open the project in your IDE:**
   ```bash
   code .
   ```

### Integrating MudBlazor

1. **Install MudBlazor NuGet package:**
   ```bash
   dotnet add package MudBlazor
   ```

2. **Configure MudBlazor in `Program.cs`:**
   ```csharp
   using MudBlazor.Services;

   var builder = WebApplication.CreateBuilder(args);

   // Add services to the container.
   builder.Services.AddRazorPages();
   builder.Services.AddServerSideBlazor();
   
   // Add MudBlazor services
   builder.Services.AddMudServices();

   var app = builder.Build();

   // Configure the HTTP request pipeline.
   if (!app.Environment.IsDevelopment())
   {
       app.UseExceptionHandler("/Error");
       app.UseHsts();
   }

   app.UseHttpsRedirection();
   app.UseStaticFiles();
   app.UseRouting();

   app.MapBlazorHub();
   app.MapFallbackToPage("/_Host");

   app.Run();
   ```

3. **Add MudBlazor CSS and JavaScript references in `_Layout.cshtml` (Blazor Server) or `index.html` (WebAssembly):**

   For Blazor Server, update `Pages/_Layout.cshtml`:
   ```html
   <!DOCTYPE html>
   <html lang="en">
   <head>
       <meta charset="utf-8" />
       <meta name="viewport" content="width=device-width, initial-scale=1.0" />
       <base href="~/" />
       <link rel="preconnect" href="https://fonts.googleapis.com" />
       <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
       <link href="https://fonts.googleapis.com/css?family=Roboto:300,400,500,700&display=swap" rel="stylesheet" />
       <link href="_content/MudBlazor/MudBlazor.min.css" rel="stylesheet" />
       <link href="css/site.css" rel="stylesheet" />
   </head>
   <body>
       @RenderBody()
       <script src="_content/MudBlazor/MudBlazor.min.js"></script>
   </body>
   </html>
   ```

4. **Add MudBlazor namespace to `_Imports.razor`:**
   ```csharp
   @using MudBlazor
   ```

5. **Update `MainLayout.razor` to use MudBlazor components:**
   ```razor
   @inherits LayoutComponentBase

   <MudThemeProvider />
   <MudDialogProvider />
   <MudSnackbarProvider />

   <MudLayout>
       <MudAppBar Elevation="1">
           <MudIconButton Icon="@Icons.Material.Filled.Menu" Color="Color.Inherit" Edge="Edge.Start" />
           <MudSpacer />
           <MudText Typo="Typo.h6">MVVM with MudBlazor Demo</MudText>
           <MudSpacer />
       </MudAppBar>
       <MudMainContent>
           <MudContainer MaxWidth="MaxWidth.Large" Class="my-4">
               @Body
           </MudContainer>
       </MudMainContent>
   </MudLayout>
   ```

## 3. MVVM Implementation with MudBlazor

Now let's implement a complete example with a Person management system.

### Creating the Model

Create a `Models` folder and add `Person.cs`:

```csharp
namespace MudBlazorMvvmDemo.Models
{
    public class Person
    {
        public int Id { get; set; }
        public string Name { get; set; } = string.Empty;
        public int Age { get; set; }
        public string Email { get; set; } = string.Empty;
        public string PhoneNumber { get; set; } = string.Empty;
    }
}
```

### Creating the ViewModel

Create a `ViewModels` folder and add `PersonViewModel.cs`:

```csharp
using MudBlazorMvvmDemo.Models;
using System.ComponentModel;
using System.Runtime.CompilerServices;

namespace MudBlazorMvvmDemo.ViewModels
{
    public class PersonViewModel : INotifyPropertyChanged
    {
        private string _name = string.Empty;
        private int _age;
        private string _email = string.Empty;
        private string _phoneNumber = string.Empty;

        // Properties with INotifyPropertyChanged implementation
        public string Name
        {
            get => _name;
            set
            {
                if (_name != value)
                {
                    _name = value;
                    OnPropertyChanged();
                }
            }
        }

        public int Age
        {
            get => _age;
            set
            {
                if (_age != value)
                {
                    _age = value;
                    OnPropertyChanged();
                }
            }
        }

        public string Email
        {
            get => _email;
            set
            {
                if (_email != value)
                {
                    _email = value;
                    OnPropertyChanged();
                }
            }
        }

        public string PhoneNumber
        {
            get => _phoneNumber;
            set
            {
                if (_phoneNumber != value)
                {
                    _phoneNumber = value;
                    OnPropertyChanged();
                }
            }
        }

        // INotifyPropertyChanged implementation
        public event PropertyChangedEventHandler? PropertyChanged;

        protected virtual void OnPropertyChanged([CallerMemberName] string? propertyName = null)
        {
            PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
        }

        // Convert ViewModel to Model
        public Person ToPerson()
        {
            return new Person
            {
                Name = Name,
                Age = Age,
                Email = Email,
                PhoneNumber = PhoneNumber
            };
        }

        // Load Model data into ViewModel
        public void LoadFromPerson(Person person)
        {
            Name = person.Name;
            Age = person.Age;
            Email = person.Email;
            PhoneNumber = person.PhoneNumber;
        }

        // Validation
        public bool IsValid()
        {
            return !string.IsNullOrWhiteSpace(Name) && Age > 0;
        }

        // Reset form
        public void Reset()
        {
            Name = string.Empty;
            Age = 0;
            Email = string.Empty;
            PhoneNumber = string.Empty;
        }
    }
}
```

### Creating the View (Razor Component)

Create `Pages/PersonManager.razor`:

```razor
@page "/person-manager"
@using MudBlazorMvvmDemo.ViewModels
@using MudBlazorMvvmDemo.Services
@inject IPersonService PersonService
@inject ISnackbar Snackbar

<PageTitle>Person Manager</PageTitle>

<MudText Typo="Typo.h3" GutterBottom="true">Person Manager</MudText>
<MudText Typo="Typo.body1" Class="mb-4">Manage people using MVVM pattern with MudBlazor</MudText>

<MudGrid>
    <MudItem xs="12" md="6">
        <MudPaper Class="pa-4">
            <MudText Typo="Typo.h5" GutterBottom="true">Add Person</MudText>
            
            <MudTextField @bind-Value="viewModel.Name" 
                          Label="Name" 
                          Variant="Variant.Outlined" 
                          Margin="Margin.Dense"
                          Required="true" />
            
            <MudNumericField @bind-Value="viewModel.Age" 
                             Label="Age" 
                             Variant="Variant.Outlined" 
                             Margin="Margin.Dense"
                             Min="0"
                             Max="150" />
            
            <MudTextField @bind-Value="viewModel.Email" 
                          Label="Email" 
                          Variant="Variant.Outlined" 
                          Margin="Margin.Dense"
                          InputType="InputType.Email" />
            
            <MudTextField @bind-Value="viewModel.PhoneNumber" 
                          Label="Phone Number" 
                          Variant="Variant.Outlined" 
                          Margin="Margin.Dense"
                          InputType="InputType.Telephone" />
            
            <MudButton Variant="Variant.Filled" 
                       Color="Color.Primary" 
                       OnClick="AddPerson"
                       Class="mt-3"
                       FullWidth="true">
                Add Person
            </MudButton>
        </MudPaper>
    </MudItem>

    <MudItem xs="12" md="6">
        <MudPaper Class="pa-4">
            <MudText Typo="Typo.h5" GutterBottom="true">Person List</MudText>
            
            @if (people.Any())
            {
                <MudList>
                    @foreach (var person in people)
                    {
                        <MudListItem>
                            <MudPaper Class="pa-3 mb-2" Elevation="2">
                                <MudGrid>
                                    <MudItem xs="8">
                                        <MudText Typo="Typo.h6">@person.Name</MudText>
                                        <MudText Typo="Typo.body2">Age: @person.Age</MudText>
                                        <MudText Typo="Typo.body2">Email: @person.Email</MudText>
                                        <MudText Typo="Typo.body2">Phone: @person.PhoneNumber</MudText>
                                    </MudItem>
                                    <MudItem xs="4" Class="d-flex align-end justify-end">
                                        <MudIconButton Icon="@Icons.Material.Filled.Edit" 
                                                       Color="Color.Primary" 
                                                       OnClick="@(() => EditPerson(person))" />
                                        <MudIconButton Icon="@Icons.Material.Filled.Delete" 
                                                       Color="Color.Error" 
                                                       OnClick="@(() => DeletePerson(person.Id))" />
                                    </MudItem>
                                </MudGrid>
                            </MudPaper>
                        </MudListItem>
                    }
                </MudList>
            }
            else
            {
                <MudAlert Severity="Severity.Info">No people added yet. Add your first person!</MudAlert>
            }
        </MudPaper>
    </MudItem>
</MudGrid>

@code {
    private PersonViewModel viewModel = new();
    private List<Models.Person> people = new();
    private bool isEditMode = false;
    private int editingPersonId = 0;

    protected override async Task OnInitializedAsync()
    {
        await LoadPeople();
    }

    private async Task LoadPeople()
    {
        people = await PersonService.GetAllAsync();
    }

    private async Task AddPerson()
    {
        if (!viewModel.IsValid())
        {
            Snackbar.Add("Please fill in all required fields", Severity.Warning);
            return;
        }

        var person = viewModel.ToPerson();

        if (isEditMode)
        {
            person.Id = editingPersonId;
            await PersonService.UpdateAsync(person);
            Snackbar.Add("Person updated successfully", Severity.Success);
            isEditMode = false;
        }
        else
        {
            await PersonService.AddAsync(person);
            Snackbar.Add("Person added successfully", Severity.Success);
        }

        viewModel.Reset();
        await LoadPeople();
    }

    private void EditPerson(Models.Person person)
    {
        viewModel.LoadFromPerson(person);
        editingPersonId = person.Id;
        isEditMode = true;
        Snackbar.Add("Edit mode activated", Severity.Info);
    }

    private async Task DeletePerson(int id)
    {
        await PersonService.DeleteAsync(id);
        Snackbar.Add("Person deleted successfully", Severity.Success);
        await LoadPeople();
    }
}
```

## 4. Data Binding in MVVM

MudBlazor provides excellent support for two-way data binding through the `@bind-Value` directive.

### Two-Way Data Binding Examples

**Text Input:**
```razor
<MudTextField @bind-Value="viewModel.Name" 
              Label="Name" 
              Variant="Variant.Outlined" />
```

**Numeric Input:**
```razor
<MudNumericField @bind-Value="viewModel.Age" 
                 Label="Age" 
                 Variant="Variant.Outlined"
                 Min="0" Max="150" />
```

**Select/Dropdown:**
```razor
<MudSelect @bind-Value="selectedValue" Label="Select Option" Variant="Variant.Outlined">
    <MudSelectItem Value="@("Option1")">Option 1</MudSelectItem>
    <MudSelectItem Value="@("Option2")">Option 2</MudSelectItem>
</MudSelect>
```

**Checkbox:**
```razor
<MudCheckBox @bind-Checked="isChecked" Label="Accept Terms" />
```

**Date Picker:**
```razor
<MudDatePicker @bind-Date="selectedDate" Label="Select Date" />
```

**Switch:**
```razor
<MudSwitch @bind-Checked="isEnabled" Label="Enable Feature" Color="Color.Primary" />
```

### Form Validation with MudBlazor

MudBlazor supports FluentValidation for comprehensive form validation:

```csharp
using FluentValidation;

public class PersonValidator : AbstractValidator<PersonViewModel>
{
    public PersonValidator()
    {
        RuleFor(x => x.Name)
            .NotEmpty().WithMessage("Name is required")
            .MaximumLength(100).WithMessage("Name cannot exceed 100 characters");

        RuleFor(x => x.Age)
            .GreaterThan(0).WithMessage("Age must be greater than 0")
            .LessThanOrEqualTo(150).WithMessage("Age must be realistic");

        RuleFor(x => x.Email)
            .EmailAddress().When(x => !string.IsNullOrEmpty(x.Email))
            .WithMessage("Invalid email format");
    }
}
```

Using validation in forms:

```razor
<MudForm @ref="form" Model="viewModel" Validation="@(validator.ValidateValue)">
    <MudTextField @bind-Value="viewModel.Name" 
                  For="@(() => viewModel.Name)"
                  Label="Name" />
    
    <MudButton OnClick="Submit">Submit</MudButton>
</MudForm>

@code {
    MudForm form;
    PersonValidator validator = new();

    private async Task Submit()
    {
        await form.Validate();
        if (form.IsValid)
        {
            // Process form
        }
    }
}
```

## 5. Dependency Injection

Dependency Injection is crucial for implementing MVVM properly, allowing for loose coupling and testability.

### Creating a Person Service

Create `Services/IPersonService.cs`:

```csharp
using MudBlazorMvvmDemo.Models;

namespace MudBlazorMvvmDemo.Services
{
    public interface IPersonService
    {
        Task<List<Person>> GetAllAsync();
        Task<Person?> GetByIdAsync(int id);
        Task AddAsync(Person person);
        Task UpdateAsync(Person person);
        Task DeleteAsync(int id);
    }
}
```

Create `Services/PersonService.cs`:

```csharp
using MudBlazorMvvmDemo.Models;

namespace MudBlazorMvvmDemo.Services
{
    public class PersonService : IPersonService
    {
        private readonly List<Person> _people = new();
        private int _nextId = 1;

        public Task<List<Person>> GetAllAsync()
        {
            return Task.FromResult(_people.ToList());
        }

        public Task<Person?> GetByIdAsync(int id)
        {
            var person = _people.FirstOrDefault(p => p.Id == id);
            return Task.FromResult(person);
        }

        public Task AddAsync(Person person)
        {
            person.Id = _nextId++;
            _people.Add(person);
            return Task.CompletedTask;
        }

        public Task UpdateAsync(Person person)
        {
            var existingPerson = _people.FirstOrDefault(p => p.Id == person.Id);
            if (existingPerson != null)
            {
                existingPerson.Name = person.Name;
                existingPerson.Age = person.Age;
                existingPerson.Email = person.Email;
                existingPerson.PhoneNumber = person.PhoneNumber;
            }
            return Task.CompletedTask;
        }

        public Task DeleteAsync(int id)
        {
            var person = _people.FirstOrDefault(p => p.Id == id);
            if (person != null)
            {
                _people.Remove(person);
            }
            return Task.CompletedTask;
        }
    }
}
```

### Registering Services in Program.cs

Update `Program.cs` to register your services:

```csharp
using MudBlazor.Services;
using MudBlazorMvvmDemo.Services;

var builder = WebApplication.CreateBuilder(args);

// Add services to the container.
builder.Services.AddRazorPages();
builder.Services.AddServerSideBlazor();

// Add MudBlazor services
builder.Services.AddMudServices();

// Register application services
builder.Services.AddSingleton<IPersonService, PersonService>();

var app = builder.Build();

// ... rest of the configuration
```

**Service Lifetimes:**
- **Singleton**: One instance for the entire application lifetime (use for stateful services)
- **Scoped**: One instance per request/circuit (recommended for most services in Blazor Server)
- **Transient**: New instance every time it's requested

For Blazor Server applications, prefer `Scoped` for most services:
```csharp
builder.Services.AddScoped<IPersonService, PersonService>();
```

## 6. Event Handling

Event handling in MVVM with MudBlazor is straightforward and powerful.

### Button Click Events

```razor
<MudButton Variant="Variant.Filled" 
           Color="Color.Primary" 
           OnClick="HandleClick">
    Click Me
</MudButton>

@code {
    private void HandleClick()
    {
        // Handle the click event
    }
}
```

### Parameterized Events

```razor
<MudButton OnClick="@(() => HandleClickWithParameter(person.Id))">
    Delete
</MudButton>

@code {
    private async Task HandleClickWithParameter(int id)
    {
        await PersonService.DeleteAsync(id);
    }
}
```

### Async Event Handlers

```razor
<MudButton OnClick="SaveDataAsync" Disabled="@isLoading">
    @if (isLoading)
    {
        <MudProgressCircular Class="ms-n1" Size="Size.Small" Indeterminate="true" />
        <MudText Class="ms-2">Saving...</MudText>
    }
    else
    {
        <MudText>Save</MudText>
    }
</MudButton>

@code {
    private bool isLoading = false;

    private async Task SaveDataAsync()
    {
        isLoading = true;
        StateHasChanged(); // Force UI update
        
        try
        {
            await Task.Delay(2000); // Simulate API call
            // Save logic here
        }
        finally
        {
            isLoading = false;
        }
    }
}
```

### Dialog Events

```razor
<MudButton OnClick="OpenDialog">Open Dialog</MudButton>

@code {
    [Inject] IDialogService DialogService { get; set; }

    private async Task OpenDialog()
    {
        var parameters = new DialogParameters { ["Person"] = viewModel };
        var dialog = await DialogService.ShowAsync<PersonDialog>("Edit Person", parameters);
        var result = await dialog.Result;

        if (!result.Canceled)
        {
            // Handle dialog result
            await LoadPeople();
        }
    }
}
```

### Input Change Events

```razor
<MudTextField @bind-Value="searchTerm" 
              Label="Search"
              OnBlur="HandleSearchBlur"
              Immediate="true"
              DebounceInterval="300"
              OnDebounceIntervalElapsed="HandleSearch" />

@code {
    private string searchTerm = "";

    private void HandleSearchBlur()
    {
        // Triggered when field loses focus
    }

    private async Task HandleSearch(string value)
    {
        // Triggered after debounce interval
        await PerformSearch(value);
    }
}
```

## 7. Styling and Themes with MudBlazor

MudBlazor offers a powerful theming system that allows you to customize the look and feel of your application.

### Creating a Custom Theme

Create a custom theme in `MainLayout.razor` or `App.razor`:

```razor
<MudThemeProvider Theme="customTheme" />
<MudDialogProvider />
<MudSnackbarProvider />

@code {
    private MudTheme customTheme = new MudTheme()
    {
        Palette = new PaletteLight()
        {
            Primary = "#1E88E5",
            Secondary = "#26A69A",
            AppbarBackground = "#1E88E5",
            Background = "#F5F5F5",
            DrawerBackground = "#FFFFFF",
            DrawerText = "rgba(0,0,0, 0.87)",
            Success = "#4CAF50"
        },
        PaletteDark = new PaletteDark()
        {
            Primary = "#90CAF9",
            Secondary = "#80CBC4",
            AppbarBackground = "#1976D2",
            Background = "#121212",
            Surface = "#1E1E1E",
            DrawerBackground = "#1E1E1E"
        },
        Typography = new Typography()
        {
            Default = new Default()
            {
                FontFamily = new[] { "Roboto", "Helvetica", "Arial", "sans-serif" },
                FontSize = ".875rem",
                FontWeight = 400,
                LineHeight = 1.43,
                LetterSpacing = ".01071em"
            },
            H1 = new H1()
            {
                FontSize = "6rem",
                FontWeight = 300,
                LineHeight = 1.167,
                LetterSpacing = "-.01562em"
            },
            H2 = new H2()
            {
                FontSize = "3.75rem",
                FontWeight = 300,
                LineHeight = 1.2,
                LetterSpacing = "-.00833em"
            }
        }
    };
}
```

### Dark Mode Toggle

Implement a dark mode toggle:

```razor
<MudThemeProvider @ref="@themeProvider" Theme="customTheme" IsDarkMode="@isDarkMode" />
<MudDialogProvider />
<MudSnackbarProvider />

<MudAppBar Elevation="1">
    <MudIconButton Icon="@Icons.Material.Filled.Menu" Color="Color.Inherit" Edge="Edge.Start" />
    <MudSpacer />
    <MudIconButton Icon="@(isDarkMode ? Icons.Material.Filled.LightMode : Icons.Material.Filled.DarkMode)" 
                   Color="Color.Inherit" 
                   OnClick="@ToggleDarkMode" />
</MudAppBar>

@code {
    private MudThemeProvider themeProvider;
    private bool isDarkMode = false;

    private void ToggleDarkMode()
    {
        isDarkMode = !isDarkMode;
    }
}
```

### Custom Component Styling

Use MudBlazor's built-in classes or custom CSS:

```razor
<!-- Using MudBlazor utility classes -->
<MudPaper Class="pa-4 ma-2" Elevation="3">
    <MudText Class="mb-2" Typo="Typo.h6">Content</MudText>
</MudPaper>

<!-- Custom CSS classes -->
<MudCard Class="custom-card">
    <MudCardContent>
        Content here
    </MudCardContent>
</MudCard>

<style>
    .custom-card {
        border-radius: 12px;
        box-shadow: 0 4px 6px rgba(0,0,0,0.1);
    }
</style>
```

### Common MudBlazor Utility Classes

- **Spacing**: `pa-{0-16}` (padding all), `ma-{0-16}` (margin all), `px-4` (padding x-axis), `my-2` (margin y-axis)
- **Display**: `d-flex`, `d-none`, `d-block`
- **Alignment**: `align-center`, `justify-center`, `align-end`, `justify-end`
- **Colors**: `primary`, `secondary`, `success`, `error`, `warning`, `info`
- **Text**: `text-center`, `text-uppercase`, `font-weight-bold`

## 8. Complete Working Example

Here's a complete working example that ties everything together:

### Full Project Structure
```
MudBlazorMvvmDemo/
├── Models/
│   └── Person.cs
├── ViewModels/
│   └── PersonViewModel.cs
├── Services/
│   ├── IPersonService.cs
│   └── PersonService.cs
├── Pages/
│   ├── PersonManager.razor
│   └── Index.razor
├── Shared/
│   └── MainLayout.razor
├── Program.cs
└── _Imports.razor
```

### Enhanced PersonManager with All Features

```razor
@page "/person-manager"
@using MudBlazorMvvmDemo.ViewModels
@using MudBlazorMvvmDemo.Services
@inject IPersonService PersonService
@inject ISnackbar Snackbar
@inject IDialogService DialogService

<PageTitle>Person Manager - MVVM Demo</PageTitle>

<MudContainer MaxWidth="MaxWidth.ExtraLarge">
    <MudText Typo="Typo.h3" Align="Align.Center" GutterBottom="true" Class="my-4">
        Person Manager
    </MudText>
    <MudText Typo="Typo.body1" Align="Align.Center" Class="mb-6">
        A complete MVVM implementation with MudBlazor
    </MudText>

    <MudGrid>
        <!-- Add/Edit Form -->
        <MudItem xs="12" md="5">
            <MudPaper Class="pa-4" Elevation="3">
                <MudText Typo="Typo.h5" GutterBottom="true">
                    @(isEditMode ? "Edit Person" : "Add New Person")
                </MudText>
                
                <MudTextField @bind-Value="viewModel.Name" 
                              Label="Name" 
                              Variant="Variant.Outlined" 
                              Margin="Margin.Dense"
                              Required="true"
                              Class="mb-2" />
                
                <MudNumericField @bind-Value="viewModel.Age" 
                                 Label="Age" 
                                 Variant="Variant.Outlined" 
                                 Margin="Margin.Dense"
                                 Min="0"
                                 Max="150"
                                 Class="mb-2" />
                
                <MudTextField @bind-Value="viewModel.Email" 
                              Label="Email" 
                              Variant="Variant.Outlined" 
                              Margin="Margin.Dense"
                              InputType="InputType.Email"
                              Class="mb-2" />
                
                <MudTextField @bind-Value="viewModel.PhoneNumber" 
                              Label="Phone Number" 
                              Variant="Variant.Outlined" 
                              Margin="Margin.Dense"
                              InputType="InputType.Telephone"
                              Class="mb-2" />
                
                <MudStack Row="true" Class="mt-3">
                    <MudButton Variant="Variant.Filled" 
                               Color="Color.Primary" 
                               OnClick="SavePerson"
                               FullWidth="true"
                               StartIcon="@Icons.Material.Filled.Save">
                        @(isEditMode ? "Update" : "Add") Person
                    </MudButton>
                    
                    @if (isEditMode)
                    {
                        <MudButton Variant="Variant.Outlined" 
                                   Color="Color.Default" 
                                   OnClick="CancelEdit"
                                   FullWidth="true">
                            Cancel
                        </MudButton>
                    }
                </MudStack>
            </MudPaper>
        </MudItem>

        <!-- Person List -->
        <MudItem xs="12" md="7">
            <MudPaper Class="pa-4" Elevation="3">
                <MudStack Row="true" AlignItems="AlignItems.Center" Class="mb-3">
                    <MudText Typo="Typo.h5">People (@people.Count)</MudText>
                    <MudSpacer />
                    <MudTextField @bind-Value="searchTerm"
                                  Placeholder="Search..."
                                  Adornment="Adornment.Start"
                                  AdornmentIcon="@Icons.Material.Filled.Search"
                                  IconSize="Size.Medium"
                                  Immediate="true"
                                  DebounceInterval="300"
                                  OnDebounceIntervalElapsed="OnSearch" />
                </MudStack>
                
                @if (filteredPeople.Any())
                {
                    <MudList>
                        @foreach (var person in filteredPeople)
                        {
                            <MudListItem>
                                <MudPaper Class="pa-3 mb-2" Elevation="2">
                                    <MudGrid>
                                        <MudItem xs="9">
                                            <MudText Typo="Typo.h6">@person.Name</MudText>
                                            <MudText Typo="Typo.body2" Color="Color.Secondary">
                                                Age: @person.Age
                                            </MudText>
                                            @if (!string.IsNullOrEmpty(person.Email))
                                            {
                                                <MudText Typo="Typo.body2">
                                                    <MudIcon Icon="@Icons.Material.Filled.Email" Size="Size.Small" /> 
                                                    @person.Email
                                                </MudText>
                                            }
                                            @if (!string.IsNullOrEmpty(person.PhoneNumber))
                                            {
                                                <MudText Typo="Typo.body2">
                                                    <MudIcon Icon="@Icons.Material.Filled.Phone" Size="Size.Small" /> 
                                                    @person.PhoneNumber
                                                </MudText>
                                            }
                                        </MudItem>
                                        <MudItem xs="3" Class="d-flex align-end justify-end">
                                            <MudIconButton Icon="@Icons.Material.Filled.Edit" 
                                                           Color="Color.Primary" 
                                                           Size="Size.Small"
                                                           OnClick="@(() => EditPerson(person))" />
                                            <MudIconButton Icon="@Icons.Material.Filled.Delete" 
                                                           Color="Color.Error" 
                                                           Size="Size.Small"
                                                           OnClick="@(() => ConfirmDelete(person.Id))" />
                                        </MudItem>
                                    </MudGrid>
                                </MudPaper>
                            </MudListItem>
                        }
                    </MudList>
                }
                else
                {
                    <MudAlert Severity="Severity.Info" Class="mt-4">
                        @(string.IsNullOrEmpty(searchTerm) 
                            ? "No people added yet. Add your first person!" 
                            : "No people found matching your search.")
                    </MudAlert>
                }
            </MudPaper>
        </MudItem>
    </MudGrid>

    <!-- Statistics Card -->
    <MudGrid Class="mt-4">
        <MudItem xs="12">
            <MudPaper Class="pa-4" Elevation="2">
                <MudText Typo="Typo.h6" GutterBottom="true">Statistics</MudText>
                <MudGrid>
                    <MudItem xs="12" sm="4">
                        <MudPaper Class="pa-3" Elevation="0" Style="background-color: var(--mud-palette-primary-lighten);">
                            <MudText Typo="Typo.h4" Color="Color.Primary">@people.Count</MudText>
                            <MudText Typo="Typo.body2">Total People</MudText>
                        </MudPaper>
                    </MudItem>
                    <MudItem xs="12" sm="4">
                        <MudPaper Class="pa-3" Elevation="0" Style="background-color: var(--mud-palette-success-lighten);">
                            <MudText Typo="Typo.h4" Color="Color.Success">@AverageAge.ToString("F1")</MudText>
                            <MudText Typo="Typo.body2">Average Age</MudText>
                        </MudPaper>
                    </MudItem>
                    <MudItem xs="12" sm="4">
                        <MudPaper Class="pa-3" Elevation="0" Style="background-color: var(--mud-palette-info-lighten);">
                            <MudText Typo="Typo.h4" Color="Color.Info">@PeopleWithEmail</MudText>
                            <MudText Typo="Typo.body2">With Email</MudText>
                        </MudPaper>
                    </MudItem>
                </MudGrid>
            </MudPaper>
        </MudItem>
    </MudGrid>
</MudContainer>

@code {
    private PersonViewModel viewModel = new();
    private List<Models.Person> people = new();
    private List<Models.Person> filteredPeople = new();
    private bool isEditMode = false;
    private int editingPersonId = 0;
    private string searchTerm = string.Empty;

    private double AverageAge => people.Any() ? people.Average(p => p.Age) : 0;
    private int PeopleWithEmail => people.Count(p => !string.IsNullOrEmpty(p.Email));

    protected override async Task OnInitializedAsync()
    {
        await LoadPeople();
    }

    private async Task LoadPeople()
    {
        people = await PersonService.GetAllAsync();
        filteredPeople = people;
    }

    private async Task SavePerson()
    {
        if (!viewModel.IsValid())
        {
            Snackbar.Add("Please fill in all required fields", Severity.Warning);
            return;
        }

        var person = viewModel.ToPerson();

        if (isEditMode)
        {
            person.Id = editingPersonId;
            await PersonService.UpdateAsync(person);
            Snackbar.Add("Person updated successfully!", Severity.Success);
            isEditMode = false;
        }
        else
        {
            await PersonService.AddAsync(person);
            Snackbar.Add($"Welcome {person.Name}!", Severity.Success);
        }

        viewModel.Reset();
        await LoadPeople();
        OnSearch(searchTerm);
    }

    private void EditPerson(Models.Person person)
    {
        viewModel.LoadFromPerson(person);
        editingPersonId = person.Id;
        isEditMode = true;
    }

    private void CancelEdit()
    {
        viewModel.Reset();
        isEditMode = false;
        editingPersonId = 0;
        Snackbar.Add("Edit cancelled", Severity.Info);
    }

    private async Task ConfirmDelete(int id)
    {
        var person = people.FirstOrDefault(p => p.Id == id);
        if (person == null) return;

        bool? result = await DialogService.ShowMessageBox(
            "Confirm Delete",
            $"Are you sure you want to delete {person.Name}?",
            yesText: "Delete", cancelText: "Cancel");

        if (result == true)
        {
            await DeletePerson(id);
        }
    }

    private async Task DeletePerson(int id)
    {
        await PersonService.DeleteAsync(id);
        Snackbar.Add("Person deleted", Severity.Success);
        await LoadPeople();
        OnSearch(searchTerm);
        
        // Clear edit form if the deleted person was being edited
        if (isEditMode && editingPersonId == id)
        {
            CancelEdit();
        }
    }

    private void OnSearch(string text)
    {
        searchTerm = text;
        if (string.IsNullOrWhiteSpace(searchTerm))
        {
            filteredPeople = people;
        }
        else
        {
            filteredPeople = people.Where(p => 
                p.Name.Contains(searchTerm, StringComparison.OrdinalIgnoreCase) ||
                p.Email.Contains(searchTerm, StringComparison.OrdinalIgnoreCase) ||
                p.PhoneNumber.Contains(searchTerm, StringComparison.OrdinalIgnoreCase)
            ).ToList();
        }
    }
}
```

## 9. Conclusion

Implementing the MVVM pattern with MudBlazor in Blazor applications provides numerous benefits for building modern, maintainable web applications:

### Key Takeaways

1. **Separation of Concerns**: MVVM cleanly separates the UI (View), presentation logic (ViewModel), and business logic (Model), making your codebase more organized and maintainable.

2. **Testability**: ViewModels can be unit tested independently from the UI, ensuring your business logic is correct without needing to render components.

3. **Reusability**: ViewModels can be shared across multiple Views, and MudBlazor components are highly reusable.

4. **Professional UI**: MudBlazor provides a rich set of Material Design components that create professional, modern interfaces without extensive custom CSS.

5. **Developer Productivity**: The combination of Blazor's component model, MVVM pattern, and MudBlazor's comprehensive component library significantly speeds up development.

6. **Data Binding**: Two-way data binding with MudBlazor components makes it easy to keep your UI in sync with your data models.

7. **Dependency Injection**: Built-in DI support in Blazor makes it easy to manage services and maintain loose coupling.

8. **Theming**: MudBlazor's powerful theming system allows you to customize the look and feel of your entire application consistently.

### Best Practices

- **Keep ViewModels Lean**: ViewModels should contain presentation logic, not business logic. Delegate business operations to services.

- **Use Dependency Injection**: Register services with appropriate lifetimes and inject them where needed.

- **Implement Validation**: Use FluentValidation or DataAnnotations for robust form validation.

- **Handle Async Operations**: Use async/await patterns properly and provide loading indicators.

- **Organize Your Code**: Maintain a clear folder structure (Models, ViewModels, Services, Components).

- **Leverage MudBlazor Components**: Use MudBlazor's rich component library instead of building custom components from scratch.

- **Consider State Management**: For complex applications, consider using state management libraries like Fluxor.

### Next Steps

To further enhance your MVVM with MudBlazor applications, consider:

1. **Adding Authentication and Authorization** using ASP.NET Core Identity
2. **Implementing API Integration** with HttpClient for backend communication
3. **Adding State Management** with Fluxor or other state management libraries
4. **Creating Custom MudBlazor Components** for reusable UI elements
5. **Implementing Advanced Validation** with FluentValidation
6. **Adding Real-time Features** with SignalR
7. **Optimizing Performance** with virtualization for large data sets
8. **Writing Unit Tests** for ViewModels and Services
9. **Implementing Error Handling** with global exception handling
10. **Adding Logging** with Serilog or other logging frameworks

### Resources

- **MudBlazor Documentation**: [mudblazor.com](https://mudblazor.com/)
- **Blazor Documentation**: [learn.microsoft.com/aspnet/core/blazor](https://learn.microsoft.com/en-us/aspnet/core/blazor)
- **MVVM Pattern**: [Microsoft MVVM Pattern Guide](https://learn.microsoft.com/en-us/dotnet/architecture/maui/mvvm)

By following this guide, you now have a solid foundation for building maintainable, testable, and visually appealing Blazor applications using the MVVM pattern with MudBlazor. Happy coding!
