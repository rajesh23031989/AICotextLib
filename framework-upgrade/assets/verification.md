# Migration Verification Checklist

## Pre-Migration Verification

- [ ] All project files identified
- [ ] Current .NET version documented
- [ ] Target .NET version confirmed
- [ ] Backup created (git branch or commit)
- [ ] Solution file validated (no stale NestedProjects entries)

## Project Files (.csproj)

- [ ] All `TargetFramework` elements updated to target version
- [ ] Documentation paths updated (if applicable)
- [ ] No hardcoded version strings remaining
- [ ] `dotnet restore` succeeds
- [ ] `dotnet build` succeeds

### Projects Verified

| Project | Updated | Build Pass |
|---------|---------|------------|
| API | [ ] | [ ] |
| Domain | [ ] | [ ] |
| Data | [ ] | [ ] |
| Metadata | [ ] | [ ] |
| Tests | [ ] | [ ] |

## Dockerfile

- [ ] Base image updated (aspnet runtime)
- [ ] SDK image updated
- [ ] `EXPOSE 8080` added
- [ ] `ENV ASPNETCORE_URLS=http://+:8080` added
- [ ] Docker build succeeds

### Dockerfile Verification

```bash
docker build -t app:migration-test .
# Expected: Build completes without errors
```

## Kubernetes Manifests

- [ ] containerPort set to 8080
- [ ] Service targetPort set to 8080
- [ ] Health probe ports updated (if applicable)
- [ ] External service port unchanged (80)

### Port Consistency Check

| Location | Expected Port | Verified |
|----------|---------------|----------|
| Dockerfile EXPOSE | 8080 | [ ] |
| Dockerfile ASPNETCORE_URLS | 8080 | [ ] |
| K8s containerPort | 8080 | [ ] |
| K8s service targetPort | 8080 | [ ] |
| K8s service port (external) | 80 | [ ] |

## CI/CD Pipelines

- [ ] All UseDotNet@2 tasks updated
- [ ] Display names updated
- [ ] .yml files checked
- [ ] .yaml files checked

### Pipelines Updated

| Pipeline | Updated | Verified |
|----------|---------|----------|
| GateBuild | [ ] | [ ] |
| VeraCode | [ ] | [ ] |
| BuildRelease | [ ] | [ ] |
| Infrastructure | [ ] | [ ] |

## NuGet Packages

- [ ] Microsoft.NET.Test.Sdk updated (test projects)
- [ ] Framework-specific packages reviewed
- [ ] Auth packages unchanged (unless requested)
- [ ] Azure Functions Worker SDK updated (if Azure Functions project)
- [ ] `Microsoft.Azure.Functions.Extensions` removed (if isolated worker model)
- [ ] `dotnet restore` succeeds

## Known Issues Check

If build fails, check for these common issues:

| Error Pattern | Issue | Fix |
|---------------|-------|-----|
| `MSB5023: A project with GUID X is listed as nested` | Stale NestedProjects in .sln | Remove orphaned GUID entries from `GlobalSection(NestedProjects)` |
| `Invalid combination of TargetFramework and AzureFunctionsVersion` | Outdated Azure Functions SDK | Upgrade `Microsoft.Azure.Functions.Worker.Sdk` to latest |
| `The current .NET SDK does not support targeting .NET X.0` | Missing SDK | Install required .NET SDK version |

## Build Verification

```bash
# Full build
dotnet restore
dotnet build --configuration Release

# Expected: Build succeeded, 0 errors
```

## Test Verification

```bash
# Run all tests
dotnet test

# Expected: All tests pass
```

## Container Verification

```bash
# Build image
docker build -t app:test .

# Run container
docker run -d -p 8080:8080 --name app-test app:test

# Test health endpoint
curl http://localhost:8080/health

# Cleanup
docker stop app-test && docker rm app-test
```

## Post-Migration Verification

- [ ] All checklist items complete
- [ ] Migration summary document created
- [ ] Rollback procedure documented
- [ ] Changes committed to feature branch
- [ ] PR created for review

## Pipeline Validation

### Feature Branch Testing

1. **Commit changes to feature branch**

```bash
git add .
git commit -m "Upgrade to .NET [version]"
git push origin feature/dotnet-upgrade
```

2. **Trigger CI pipeline**
   - Create PR to trigger validation build
   - Or manually queue pipeline on feature branch

3. **Verify pipeline output**
   - [ ] SDK version shows target version in logs
   - [ ] `dotnet restore` succeeds
   - [ ] `dotnet build` succeeds
   - [ ] All unit tests pass
   - [ ] Code coverage unchanged or improved
   - [ ] No new warnings introduced

### Pipeline Log Verification

Check build logs for:

```text
Installing .NET version 10.x
...
Restore completed in X seconds
Build succeeded.
    0 Warning(s)
    0 Error(s)
```

### Pre-Merge Checklist

- [ ] Pipeline passes on feature branch
- [ ] PR reviewed and approved
- [ ] No merge conflicts with main
- [ ] Deployment to dev/test environment succeeds
- [ ] Smoke tests pass in deployed environment

## Sign-off

| Role | Name | Date | Approved |
|------|------|------|----------|
| Developer | | | [ ] |
| Reviewer | | | [ ] |
| Tech Lead | | | [ ] |
