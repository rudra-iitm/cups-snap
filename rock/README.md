# The OpenPrinting CUPS Rock

Complete CUPS printing stack in a OCI image

[CUPS in the Dockerhub]() - Link to be updated

[All from OpenPrinting in the Dockerhub]() - Link to be updated

## Introduction

This is a complete printing stack in a OCI image. It contains not only CUPS but also cups-filters, Ghostscript, and Poppler (the two latter as PostScript and PDF interpreters). This is everything (except printer-model-specific drivers) which is needed for printing.

This OCI image is designed for the following use cases:

1. Providing a printing stack for a many immutable linux distribution.
2. Providing a printing stack for a classic Linux system, installing the OCI image instead of the system's usual printing packages.

Note that this OCI image is still under development and therefore there are probably still many bugs.

## Documentation
- `README.md`: This file contains introduction,  instructions and details about the setup, usage, and configuration of the CUPS Rock.
- `TESTING.md`: This file contains detailed steps for testing the CUPS Rock functionality.

## Install from Docker Hub
### Prerequisites

1. **Docker Installed**: Ensure Docker is installed on your system. You can download it from the [official Docker website](https://www.docker.com/get-started).

- `cups-ipp-utils`
- `cups-client`

  Install the required packages on docker-host using the following commands:

  ```sh
  sudo apt-get update
  sudo apt-get install cups-ipp-utils cups-client -y
  ```

### Step-by-Step Guide

#### 1. Pull the CUPS Docker image:

The first step is to pull the CUPS Docker image from Docker Hub. This image contains a complete printing stack inside an OCI image.
```sh
sudo docker pull openprinting/cups
```

#### 2. Start the CUPS Container
Open another terminal session and start the CUPS container with the following environment variables:

```sh
CUPS_ADMIN=<cups_username>
CUPS_PASSWORD=<cups_password>
```

##### Run the following Docker command to run the cups image:
```sh
sudo docker run --rm -d --name cups -p <port>:631 \
    -e CUPS_ADMIN="${CUPS_ADMIN}" -e CUPS_PASSWORD="${CUPS_PASSWORD}" \
    openprinting/cups:latest
```

## Setting Up and Running CUPS locally

### Prerequisites

1. **Docker Installed**: Ensure Docker is installed on your system. You can download it from the [official Docker website](https://www.docker.com/get-started).

2. **Rockcraft**: Rockcraft should be installed. You can install Rockcraft using the following command:
```sh
sudo snap install rockcraft --classic
```

3. **Skopeo**: Skopeo should be installed to compile `.rock` files into Docker images. <br>
**Note**: It comes bundled with Rockcraft.

### Step-by-Step Guide

#### 1. Build CUPS rock:

The first step is to build the Rock from the `rockcraft.yaml`. This image will contain all the configurations and dependencies required to run CUPS.

Open your terminal and navigate to the directory containing your `rockcraft.yaml`, then run the following command:

```sh
rockcraft pack -v
```

#### 2. Compile to Docker Image:

Once the rock is built, you need to compile docker image from it.

```sh
sudo rockcraft.skopeo --insecure-policy copy oci-archive:<rock_image> docker-daemon:cups:latest
```

#### Run the CUPS Docker Container:

```sh
sudo docker run --rm -d --name cups -p <port>:631 \
    -e CUPS_ADMIN="${CUPS_ADMIN}" -e CUPS_PASSWORD="${CUPS_PASSWORD}" \
    cups:latest
```

## Note: 
- `CUPS_ADMIN` and `CUPS_PASSWORD` are administrative credentials for accessing CUPS.
- If credentials are not provided, you can view the randomly generated administrative credentials for your container using the following command:
    ```sh
    sudo docker exec cups cat /etc/cups/cups-credentials
    ```

### Accessing the CUPS Web Interface
- The CUPS web interface can be accessed at `http://localhost:<port>` to manage printers and check job statuses.

## CUPS Commands
To use use the cups's command line utilities acting on the CUPS image, proceed the commands with following format:

1. Use Docker's -u flag to specify the CUPS_ADMIN user
```sh
sudo docker exec -u "${CUPS_ADMIN}" cups <command>
```
Example:
To add a printer from inside the container:
```sh
sudo docker exec -u "${CUPS_ADMIN}" cups lpadmin -p <printer>
```

2. Use `cups-client` on Docker-host to execute CUPS commands:s
```sh
CUPS_SERVER=localhost:<port> <command>
```

Example:
To check the print status:
```sh
CUPS_SERVER=localhost:<port> lpstat -W completed
```
**Note to use cups administrative task pass -U flag with lpadmin command**