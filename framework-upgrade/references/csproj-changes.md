# Project File Changes (.csproj)

## Target Framework Update

### Before
```xml
<TargetFramework>net8.0</TargetFramework>
```

### After
```xml
<TargetFramework>net10.0</TargetFramework>
```

## Documentation Path Updates

If projects generate XML documentation, update paths:

### Before
```xml
<DocumentationFile>bin\$(Configuration)\net8.0\MyProject.xml</DocumentationFile>
```

### After
```xml
<DocumentationFile>bin\$(Configuration)\net10.0\MyProject.xml</DocumentationFile>
```

## Search Pattern

Find all instances with:

```
net8.0
```

Replace with:

```
net10.0
```

## Files to Update

Typical ASP.NET Core solution structure:
- `MyProject.API/MyProject.API.csproj` - Main API
- `MyProject.Domain/MyProject.Domain.csproj` - Domain layer
- `MyProject.Data/MyProject.Data.csproj` - Data layer
- `MyProject.MetaData/MyProject.MetaData.csproj` - DTOs/Models
- `Tests/Tests.csproj` - Test project

## Verification

After update, verify:

```bash
dotnet restore
dotnet build
```

Expected output: No target framework errors
