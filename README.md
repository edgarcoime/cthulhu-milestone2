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
3. Filemanager (Not publicly available since Gateway is the only entry point)
4. Authentication (Not publicly available since Gateway is the only entry point)

## Docker

All services can also be run using Docker. This allows me the flexibility to later run orchestration through docker-compose or Kubernetes. This is still a work in progress and not fully working so will add onto further milestones. Only one working 100% is RabbitMQ.

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
