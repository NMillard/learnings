## Containers
A container is spun up of an image, and will create a new file system layer that has read/write access

## Volumes
Attach volume to container using `-v`

Volumes can be attached inside the docker virtual disk or on the acutal computers disk.
Creating a volume for container: `-v path`
Mount volume host path to container path `-v /c/path:containerPath`
Clean up when removing containers using `docker rm -v container-name`

Should try opt for the newer `--mount` flag instead of `-v`.

Example of host path to container path mapping
`--mount type=volume,source=/c/path,target=/var/app`

## The Dockerfile
File to build images. Provide the step-by-step build and publish process.  
See example of how to create an aspnetcore docker image below.

```dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:6.0 AS base
WORKDIR /app
EXPOSE 80
EXPOSE 443

FROM mcr.microsoft.com/dotnet/sdk:6.0 AS build
WORKDIR /app
COPY *.csproj .
RUN dotnet restore

COPY . .
RUN dotnet build -c Release -o /app/build

FROM build AS publish
RUN dotnet publish -c Release -o /app/publish

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "DockerizedAspNetCore.dll"]

```

## Container registry
A place to store, share and pull built images.  

Push an image with `docker image push <hub-id>/<repo-name>:<version>`.

## The docker-compose.yml file
Define service orchestration

## Docker networking
- Container Network Model (CNM)
    Sandbox (Namespace): isolated area of OS
    Endpoint: Network interface
    Network: Connected endpoints
- Libnetwork
    Controle plane
- Drivers
    Data plane

Multi-host networking with overlay driver is used to connect multiple computer to one network.
To create a swarm, ports 4789/UDP, 7946/TCP/UDP, and 2377/TCP must be open

## Docker on Windows Server
Install docker using PowerShell