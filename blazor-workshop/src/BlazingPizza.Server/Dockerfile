# Build stage
FROM mcr.microsoft.com/dotnet/sdk:7.0 AS base
WORKDIR /src

# Copy only necessary backend files
COPY ["Directory.Build.props","."]
COPY src/BlazingPizza.Server/BlazingPizza.Server.csproj src/BlazingPizza.Server/
COPY src/BlazingPizza.Shared/BlazingPizza.Shared.csproj src/BlazingPizza.Shared/
COPY ["src/BlazingComponents/BlazingComponents.csproj", "src/BlazingComponents/"]
COPY ["src/BlazingPizza.ComponentsLibrary/BlazingPizza.ComponentsLibrary.csproj", "src/BlazingPizza.ComponentsLibrary/"]
COPY ["src/BlazingPizza.Client/BlazingPizza.Client.csproj", "src/BlazingPizza.Client/"]



# Initial restore
RUN dotnet restore "src/BlazingPizza.Server/BlazingPizza.Server.csproj"

# Stage 2: Build
FROM mcr.microsoft.com/dotnet/sdk:7.0 AS build
WORKDIR /src
COPY --from=base /src .
COPY . .
# Copy ALL source files (including Program.cs)
COPY ["src/BlazingPizza.Server/", "src/BlazingPizza.Server/"]
COPY ["src/BlazingPizza.Shared/", "src/BlazingPizza.Shared/"]

# Build and publish
WORKDIR "/src/src/BlazingPizza.Server"
RUN dotnet publish -c Release -o /app/publish \
    /p:UseAppHost=false

# Stage 3: Runtime
FROM mcr.microsoft.com/dotnet/aspnet:7.0 AS final
WORKDIR /app
COPY --from=build /app/publish .

# Install SQLite dependencies
RUN apt-get update && \
    apt-get install -y --no-install-recommends libsqlite3-dev && \
    rm -rf /var/lib/apt/lists/*

# Create data directory
RUN mkdir -p /app/Data

ENTRYPOINT ["dotnet", "BlazingPizza.Server.dll"]
