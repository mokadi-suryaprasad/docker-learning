# Dockerfiles for .NET (ASP.NET Core)

## When to use
- **Single‑stage**: you have `dotnet publish` output.
- **Multi‑stage**: build + runtime in one Dockerfile.

### Layout
```
app/
├─ MyApp.sln
├─ MyApp/
└─ Dockerfile
```

---

## Single‑Stage (pre‑published)
```dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:8.0
WORKDIR /app
COPY ./publish/ .
EXPOSE 8080
ENTRYPOINT ["dotnet","MyApp.dll"]
```

---

## Multi‑Stage
```dockerfile
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src
COPY *.sln .
COPY MyApp/*.csproj MyApp/
RUN dotnet restore
COPY . .
RUN dotnet publish MyApp -c Release -o /out

FROM mcr.microsoft.com/dotnet/aspnet:8.0
WORKDIR /app
COPY --from=build /out .
EXPOSE 8080
ENTRYPOINT ["dotnet","MyApp.dll"]
```
