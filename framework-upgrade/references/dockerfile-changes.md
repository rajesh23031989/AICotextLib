# Dockerfile Changes

## Base Image Updates

### Option 1: Microsoft Container Registry (Recommended)

#### Before (.NET 8)
```dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS base
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
```

#### After (.NET 10)
```dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:10.0 AS base
FROM mcr.microsoft.com/dotnet/sdk:10.0 AS build
```

### Option 2: Private ACR

#### Before (.NET 8)
```dockerfile
FROM acrcmfgmaster.azurecr.io/aspnetcore-runtime:8.0 AS base
FROM acrcmfgmaster.azurecr.io/dotnetcore-sdk:8.0 AS build
```

#### After (.NET 10)
```dockerfile
FROM acrcmfgmaster.azurecr.io/aspnetcore-runtime:10.0 AS base
FROM acrcmfgmaster.azurecr.io/dotnetcore-sdk:10.0 AS build
```

## Debian Linux distribution to an Ubuntu distro

Switching to the .Net 10 default image (Ubuntu) may introduce some syntax errors in the docker file.

```dockerfile
RUN addgroup --system dotnetappgroup && \
    adduser --system --ingroup dotnetappgroup --shell /bin/sh dotnetappuser && \
    mkdir -p /local/ && \
    chown dotnetappuser:dotnetappgroup /local && \
    chown dotnetappuser:dotnetappgroup /app

USER dotnetappuser
```

Ubuntu Migration

Change it to:

```dockerfile
RUN groupadd -g 1001 dotnetappgroup && \
    useradd -u 1001 -g dotnetappgroup -d /app -s /sbin/nologin -c "Docker image user" dotnetappuser && \
    chown -R dotnetappuser:dotnetappgroup /app
```

https://learn.microsoft.com/en-us/dotnet/core/compatibility/10?toc=%2Faspnet%2Fcore%2Ftoc.json&bc=%2Faspnet%2Fcore%2Fbreadcrumb%2Ftoc.json&view=aspnetcore-10.0#containers

https://learn.microsoft.com/en-us/dotnet/core/compatibility/containers/10.0/default-images-use-ubuntu

## Port Configuration

.NET 8+ uses port **8080** by default (not 5000).

### Add to Base Stage
```dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:10.0 AS base
WORKDIR /app
EXPOSE 8080
ENV ASPNETCORE_URLS=http://+:8080
```

## Complete Dockerfile Example

```dockerfile
# Base stage
FROM mcr.microsoft.com/dotnet/aspnet:10.0 AS base
WORKDIR /app
EXPOSE 8080
ENV ASPNETCORE_URLS=http://+:8080

# Build stage
FROM mcr.microsoft.com/dotnet/sdk:10.0 AS build
WORKDIR /src
COPY ["MyProject.API/MyProject.API.csproj", "MyProject.API/"]
COPY ["MyProject.Domain/MyProject.Domain.csproj", "MyProject.Domain/"]
COPY ["MyProject.Data/MyProject.Data.csproj", "MyProject.Data/"]
RUN dotnet restore "MyProject.API/MyProject.API.csproj"
COPY . .
WORKDIR "/src/MyProject.API"
RUN dotnet build "MyProject.API.csproj" -c Release -o /app/build

# Publish stage
FROM build AS publish
RUN dotnet publish "MyProject.API.csproj" -c Release -o /app/publish

# Final stage
FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "MyProject.API.dll"]
```

## Verification

```bash
docker build -t myapp:net10 .
docker run -p 8080:8080 myapp:net10
curl http://localhost:8080/health
```
