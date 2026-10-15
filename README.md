# DevOps Winter Arc — Day 02
## Dockerfile Deep Dive

### Project Overview
This project builds a Node.js HTTP API using Docker.
The API supports a greeting endpoint and a health-check endpoint.

### Files
- Dockerfile: Builds the Node.js API image.
- Dockerfile.greeter: Demonstrates CMD and ENTRYPOINT.
- package.json: Defines the Node.js project.
- server.js: Implements the HTTP API.
- .dockerignore: Excludes unnecessary files from the build context.
- evidence/: Contains screenshots of the experiments.

### Dockerfile Explanation
- FROM node:20-alpine: Uses a lightweight Node.js base image.
- WORKDIR /app: Sets the working directory inside the image.
- ARG APP_VERSION=1.0: Defines a build-time argument.
- ENV PORT=3000 APP_ENV=dev APP_VERSION=$APP_VERSION: Sets image environment variables.
- COPY package*.json ./: Copies dependency manifests first.
- RUN npm install --omit=dev: Installs production dependencies.
- COPY . .: Copies application files into the image.
- EXPOSE 3000: Documents the port used by the application.
- USER node: Runs the application as the non-root node user.
- CMD ["node", "server.js"]: Defines the default application command.

### Build
docker build -t demo-api:v1 --build-arg APP_VERSION=1.0 .

### Run
docker run -d --name api -p 3000:3000 demo-api:v1

### Test Endpoints
curl http://localhost:3000/
curl http://localhost:3000/health

The health endpoint should return: ok

### Runtime Environment Variables
docker rm -f api
docker run -d --name api -p 3000:3000 -e GREETING="Hi Winter Arc" -e APP_ENV=prod demo-api:v1
curl http://localhost:3000/

### Environment File
Create a local .env file containing GREETING and APP_ENV values.
Run the container with:
docker run -d --name api -p 3000:3000 --env-file .env demo-api:v1

Do not commit .env or secrets to GitHub.

### Docker Layer Caching
The Dockerfile copies package manifests before installing dependencies.
Changing only server.js should allow the dependency-install layer to be reused.
Changing package.json invalidates the manifest-copy layer and subsequent layers.

Build history and image configuration can be inspected with:
docker history demo-api:v3
docker image inspect demo-api:v3

### CMD vs ENTRYPOINT
Dockerfile.greeter uses ENTRYPOINT to define echo Hello and CMD to provide the default argument World.

docker build -f Dockerfile.greeter -t greeter:local .
docker run --rm greeter:local
docker run --rm greeter:local DevOps

Expected output:
Hello World
Hello DevOps

### Reflection
- CMD defines the default command or arguments.
- ENTRYPOINT defines the executable.
- COPY copies files into the image; ADD also supports additional features such as archive extraction.
- ARG is intended for build-time configuration; ENV defines environment variables available in the image and container.
- Do not store secrets in ARG or ENV.
- EXPOSE documents a container port but does not publish it.
- The -p option publishes a port to the host.
- Copying dependency manifests before source code improves layer-cache reuse.
- Exec-form CMD supports proper signal handling.
- .dockerignore reduces build context and prevents excluded files from being sent to the builder.

### Evidence
Screenshots are stored in the evidence/ directory.
