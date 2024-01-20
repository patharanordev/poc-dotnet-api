# POC DotNet API

## Docker container

### Developing ASP.NET Core Applications with Docker over HTTPS

Assume our credential is `123456` :

```sh
# Linux
export CREDENTIAL_PLACEHOLDER=123456

# Window
# set CREDENTIAL_PLACEHOLDER=123456
```

Generate cert and configure local machine:

> ---
> NOTE: `DOTNET_ROOT` is `C:\Program Files\dotnet`
>
> ---

```sh
# Linux
dotnet dev-certs https -ep ~/.aspnet/https/poc-dotnet-api.pfx -p $CREDENTIAL_PLACEHOLDER

# Window (requires full-file path)
# dotnet dev-certs https -ep C:/Users/patha/.aspnet/https/poc-dotnet-api.pfx -p %CREDENTIAL_PLACEHOLDER%

dotnet dev-certs https --trust
```

Configure application secrets, for the certificate:

```sh
dotnet user-secrets init -p poc-dotnet-api.csproj

# Linux
dotnet user-secrets -p poc-dotnet-api.csproj set "Kestrel:Certificates:Development:Password" "$CREDENTIAL_PLACEHOLDER"

# Window
# dotnet user-secrets -p poc-dotnet-api.csproj set "Kestrel:Certificates:Development:Password" %CREDENTIAL_PLACEHOLDER%
```

### Docker compose

You can apply above config to docker compose by set environment variable below:

- **ASPNETCORE_Kestrel__Certificates__Default__Password** — passes the password to the container's `ASPNETCORE_Kestrel__Certificates__Default__Password` environment variable.
- **ASPNETCORE_Kestrel__Certificates__Default__Path** — stores the inner location of the certificate in an environment variable.

```yml
    # ...

    environment:
      - ASPNETCORE_ENVIRONMENT=Development
      - ASPNETCORE_URLS=https://+:443;http://+:80
      - ASPNETCORE_Kestrel__Certificates__Default__Password=123456
      - ASPNETCORE_Kestrel__Certificates__Default__Path=/root/.aspnet/https/poc-dotnet-api.pfx
```

Then set volume :

- `/root/.aspnet/https` — mounts the folder where the certificate was created.

```yml
    # ...

    # Linux
    volumes:
      - ~/.aspnet/https:/root/.aspnet/https:ro
      - ~/.microsoft/usersecrets:/root/.microsoft/usersecrets:ro

    # Window
    # volumes:
    #  - $env:USERPROFILE\.aspnet\https:/root/.aspnet/https:ro
    #  - $env:APPDATA\microsoft\UserSecrets:/root/.microsoft/usersecrets:ro
```

Or just :

```sh
# Linux
docker compose -f docker-compose.yml down -v;
docker compose -f docker-compose.yml up --build;

# Window
# docker compose -f docker-compose.win.yml down -v;
# docker compose -f docker-compose.win.yml up --build;
```

Enjoy `https://localhost/WeatherForecast` :

```json
[{
  "date":"2024-01-20",
  "temperatureC":22,
  "temperatureF":71,
  "summary":"Sweltering"
}]
```
