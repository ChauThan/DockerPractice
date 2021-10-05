# Docker Practice

In this practice, we can run container with HTTPS by docker and docker-compose.

## Prerequisite:
- C# IDE (VS Code, Visual Studio or Rider).
- .NET Core SDK 5
- Docker
- pwsh

## Prepare a certificate.
In order to make a container run with HTTPS, we have to create a certificate for it. Note that we are developing, so this cert works as locahost certificate.
- Create certificate by dotnet tool. Remember set password for it. In this practice, I used `pa55w0rd!`
```shell
dotnet dev-certs https -ep $env:USERPROFILE\.aspnet\https\docker-practice-api.pfx -p pa55w0rd!
```

- Then trust it.
```shell
dotnet dev-certs https --trust
```

- Create User secret
```shell
dotnet user-secrets init
```

- Add certificate to project
```shell
dotnet user-secrets set "Kestrel:Certificates:Development:Password" "pa55w0rd!"
```

## Build docker image
- Open terminal and set current directory to platform folder then run.
```shell
docker build -t chauthan/dockerpractice .
```

## Run container with HTTPS
In order to run with HTTPS, the environment needs 2 variables.
 - `ASPNETCORE_Kestrel__Certificates__Default__Path` points to the `.pfx` we created.
 - `ASPNETCORE_Kestrel__Certificates__Default__Password` set the password we used for certicate.

The command will be
```shell
docker run -d --rm `
  -p 8080:80 -p 8081:443 `
  -e ASPNETCORE_URLS="https://+;http://+" `
  -e ASPNETCORE_HTTPS_PORT=8081 `
  -e ASPNETCORE_ENVIRONMENT=Development `
  -e ASPNETCORE_Kestrel__Certificates__Default__Password="pa55w0rd!" `
  -e ASPNETCORE_Kestrel__Certificates__Default__Path=/root/.aspnet/https/docker-practice-api.pfx `
  -v $env:USERPROFILE\.aspnet\https:/root/.aspnet/https `
  chauthan/dockerpractice
```

We also parse it to compose file.
```yaml
version: "3.9"
services: 
  api:
    image: "chauthan/dockerpractice"
    ports:
      - "8080:80"
      - "8081:443"
    environment:
      ASPNETCORE_URLS: "https://+;http://+"
      ASPNETCORE_HTTPS_PORT: "8081"
      ASPNETCORE_ENVIRONMENT: "Development"
      ASPNETCORE_Kestrel__Certificates__Default__Password: "pa55w0rd!"
      ASPNETCORE_Kestrel__Certificates__Default__Path: /root/.aspnet/https/docker-practice-api.pfx
    volumes:
      - ${USERPROFILE}\.aspnet\https:/root/.aspnet/https/             
```
Then run
```shell
docker-compose up --build
```

## References
https://docs.microsoft.com/en-us/aspnet/core/security/docker-https?view=aspnetcore-5.0
https://docs.microsoft.com/en-us/aspnet/core/security/docker-compose-https?view=aspnetcore-5.0