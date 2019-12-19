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
File to build images

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