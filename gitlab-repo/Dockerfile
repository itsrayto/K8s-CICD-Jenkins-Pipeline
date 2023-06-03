FROM mcr.microsoft.com/dotnet/aspnet:7.0 AS base
WORKDIR "/app"
EXPOSE 80

FROM mcr.microsoft.com/dotnet/sdk:7.0 AS build
WORKDIR "/src"

# This copies the project file TheLegend.csproj from the local dir to the /src/TheLegend/ dir inside the container.
COPY ["TheLegend/TheLegend.csproj", "TheLegend/"]

# restores the dependencies specified in the project file.
RUN dotnet restore "TheLegend/TheLegend.csproj"

COPY . .
WORKDIR "/src/TheLegend"

# builds(compiles) the app in Release mode and outputs the build artifacts to the /app/build dir.
RUN dotnet build "TheLegend.csproj" -c Release -o /app/build

FROM build AS publish
# publishes the app in Release mode and outputs the published files to the /app/publish dir.
RUN dotnet publish "TheLegend.csproj" -c Release -o /app/publish

FROM base AS final
WORKDIR "/app"
COPY --from=publish /app/publish .
# specifies the command to run when the container starts - runs the ASP.NET Core app.
ENTRYPOINT ["dotnet", "TheLegend.dll"]
