# Cthulhu

## What is CTHULHU?

CTHULHU is an anonymous, file sharing platform that lets anyone upload up to 1 GB of files without an account and share a secure URL that expires after 48 hours. Authorized users can extend retention up to 14 days, manage, and delete their uploads on demand, and optionally password-protect shared content. Built on a microservices architecture with a focus on scalability, security, and user privacy.

## <a name="toc">Table of Contents</a>

- [What is CTHULHU](#what-is-cthulhu)
- [Table of contents](#toc)
- [How to Populate Submodules](#submodule)
- [Prerequisites](#prerequisites)
- [Development Environment (RECOMMENDED)](#development-environment)
  - [1. Setup RabbitMQ](#1-setup-rabbitmq)
  - [2. Development Startup Process](#dev-process)
  - [3. Test application](#3-test-application)
- [Docker (Work in Progress)](#docker)
  - [RabbitMQ](#rabbitmq)
  - [Client](#client)
  - [Gateway](#gateway)
  - [Filemanager](#filemanager)

## Prerequisites

- Golang 1.25.3 - [download](https://go.dev/doc/install)
- Node 23.11.0 - [download](https://nodejs.org/en/download)
- Docker 28.4.0 (Any recent version is fine) - [download](https://docs.docker.com/engine/install/)
- Make (Most UNIX based systems have it otherwise install using package manager)

## How to Populate Submodules

If you have cloned this project as a checkpoints with associated submodules then when cloned the respective services will be empty. The following commands will need to be run to populate the folders with its associated files according to its commit.

```bash
# Initialize submodules with its respective commit hash
git submodule init

# Clone all code recursively up to that init hash
git submodule update --recursive
```

## Development Environment

Start by navigating to the root of the project.

### 1. Setup RabbitMQ

```bash
# Navigate to rabbitmq folder
cd ./rabbitmq

# Build rabbitmq
docker build -t cthulhu-rabbitmq .

# Run the container with environment vars
docker run -d --name cthulhu-rabbitmq \
  -p 5672:5672 -p 15672:15672 -p 25672:25672 \
  cthulhu-rabbitmq

# Check if container is running
docker ps
```

Can look at the admin platform at [dashboard](http://localhost:15672).

The default credentials are:

- username: _guest_
- password: _guest_

### <a name="dev-process">2. Initialize by Starting Services One by One</a>

The Process to start up the project is a little more involved this time. It involves navigating to each service and running `make dev` to spin up each one individually. This of course assumes that you have the required `.env` files. I cannot post them here because it has secret codes to AWS and oauth secrets.

Spin up the services in the following order:

- RabbitMQ Message Broker (./rabbitmq) (Previous step should be run using docker)
- Auth Service (./auth)
- Filemanager Service (./filemanager)
- Gateway Service (./gateway)
- Client Frontend Web service (./client)

Navigate to each respective folder and run the `make dev` command.

```bash
# Navigate inside the service folder with its own Makefile
cd <service_folder>

# Start dev environment
make dev
```

### 3. Test application

Test the main resources of the application.

1. Client (Navigate to [localhost:3000](http://localhost:3000))
2. Gateway (Navigate to [localhost:7777](http://localhost:7777))
3. Filemanager (Not fully implemented yet)

## Docker

All services can also be run using Docker. This allows me the flexibility to later run orchestration through docker-compose or Kubernetes. Here are some ways the services can be run through Docker.

### RabbitMQ

```bash
cd ./rabbitmq
docker build -t cthulhu-rabbitmq

# Run
docker run -d \
  --name cthulhu-rabbitmq \
  -p 5672:5672 \
  -p 15672:15672 \
  -p 25672:25672 \
  cthulhu-rabbitmq
```

### Client

```bash
# Build Docker image
cd ./client
docker build -t cthulhu-client .

docker run -p 3000:3000 \
  -e NEXT_PUBLIC_API_URL=http://localhost:8985 \
  -e NEXT_PUBLIC_APP_NAME=Cthulhu \
  -e NEXT_PUBLIC_VERSION=0.1.0 \
  -e ROOT_DOMAIN=http://localhost:3000 \
  cthulhu-client
```

### Gateway

Gateway requires a lot of environment variables. Because of requirements with common it needs to be built from the root of the project.

Required Variables:

- JWT_SECRET - Required for JWT token signing
- GITHUB_CLIENT_ID - Required for OAuth
- GITHUB_CLIENT_SECRET - Required for OAuth
- S3_ACCESS_KEY_ID - Required if using S3
- S3_SECRET_ACCESS_KEY - Required if using S3

```bash
# Build docker image
cd /project_root
docker build -f gateway/MAIN/Dockerfile -t cthulhu-gateway .

# Run with environment variable overrides
docker run -p 7777:7777 \
  -e JWT_SECRET=your-secret-key-here \
  -e GITHUB_CLIENT_ID=your-github-client-id \
  -e GITHUB_CLIENT_SECRET=your-github-client-secret \
  -e S3_ACCESS_KEY_ID=your-s3-access-key \
  -e S3_SECRET_ACCESS_KEY=your-s3-secret-key \
  -e CORS_ORIGIN=http://localhost:3000 \
  -e AMQP_HOST=rabbitmq \
  -e S3_ENDPOINT=http://localstack:4566 \
  -v $(pwd)/gateway/MAIN/db:/app/db \
  cthulhu-gateway

```

### Filemanager

Work in progress not fully implemented and working with Docker yet

### Testing functionality

You can use Postman or curl. Right now the gateway is expecting the files in formdata form.

```bash
cd <root of project>

curl --location 'http://localhost:7777/files/upload' \
  --form 'file=@./testfiles/test1.txt' \
  --form 'file=@./testfiles/test2.txt' \
  --form 'file=@./testfiles/test3.txt' \
  --form 'file=@./testfiles/test_med.pdf'
```
