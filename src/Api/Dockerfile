# Get base SDK Image from Microsoft
FROM mcr.microsoft.com/dotnet/sdk:5.0 AS build
WORKDIR /app

# Copy the csproj file and restore any dependencies.
COPY ["*.csproj", "./"]
RUN dotnet restore 

# Copy the project files and build & publish release.
COPY . ./
RUN dotnet publish  -c Release -o out

# Generate runtime image.
FROM mcr.microsoft.com/dotnet/aspnet:5.0
WORKDIR /app
EXPOSE 80
COPY --from=build /app/out .
ENTRYPOINT ["dotnet", "Api.dll"]
