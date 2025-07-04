# Always build on amd64, since .NET doesn't support QEMU-based cross-compilation

# Stage 1: Build the app
FROM mcr.microsoft.com/dotnet/sdk:7.0 AS build

WORKDIR /source

# Copy and restore dependencies
COPY *.csproj .
RUN dotnet restore

# Copy the remaining source and publish
COPY . .
RUN dotnet publish -c Release -o /app --self-contained false --no-restore

# Stage 2: Create the runtime image
FROM mcr.microsoft.com/dotnet/runtime:7.0

WORKDIR /app
COPY --from=build /app .

ENTRYPOINT ["dotnet", "Worker.dll"]
