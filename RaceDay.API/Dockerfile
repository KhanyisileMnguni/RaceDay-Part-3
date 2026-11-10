# syntax=docker/dockerfile:1
#
# Dockerfile for the RaceDay API (ASP.NET Core 10 Web API).
# Build from the repository root:
#   docker build -f RaceDay.API/Dockerfile -t raceday-api .

# ---------- Stage 1: build and publish ----------
FROM mcr.microsoft.com/dotnet/sdk:10.0 AS build
WORKDIR /src

# Copy only the project file first so the NuGet restore layer is cached
# until the packages change.
COPY RaceDay.API/RaceDay.API.csproj RaceDay.API/
RUN dotnet restore RaceDay.API/RaceDay.API.csproj

# Copy the remaining source code and publish.
COPY RaceDay.API/ RaceDay.API/
RUN dotnet publish RaceDay.API/RaceDay.API.csproj \
    -c Release \
    -o /app/publish \
    --no-restore

# ---------- Stage 2: small runtime image ----------
FROM mcr.microsoft.com/dotnet/aspnet:10.0 AS final
WORKDIR /app

# ASP.NET Core listens on 8080 inside the container (the default for the
# aspnet:10.0 image). The environment variables below are overridden by
# docker-compose.yml or the docker run command when secrets are needed.
ENV ASPNETCORE_URLS=http://+:8080 \
    ASPNETCORE_ENVIRONMENT=Production

EXPOSE 8080
COPY --from=build /app/publish .

# Run as the built-in non-root user for better security.
USER $APP_UID
ENTRYPOINT ["dotnet", "RaceDay.API.dll"]
