### Building and running your application

When you're ready, start your application by running:
`docker compose up --build`.

Your application will be available at http://localhost:8080. (/weather/forecast for this simple API)

### Deploying your application to the cloud

First, build your image, e.g.: `docker build -t myapp .`. (-t is the tag/name of the resulting image, and the period means docker should look for the dockerfile in the current folder)
If your cloud uses a different CPU architecture than your development
machine (e.g., you are on a Mac M1 and your cloud provider is amd64),
you'll want to build the image for that platform, e.g.:
`docker build --platform=linux/amd64 -t myapp .`.

Then, push it to your registry, e.g. `docker push myregistry.com/myapp`.

Consult Docker's [getting started](https://docs.docker.com/go/get-started-sharing/)
docs for more detail on building and pushing.

### References
* [Docker's .NET guide](https://docs.docker.com/language/dotnet/)
* The [dotnet-docker](https://github.com/dotnet/dotnet-docker/tree/main/samples)
  repository has many relevant samples and docs.


### Learning to deploy a dockerized .NET Core API in Elastic Beanstalk
- First step is to have a working .NET API that runs locally;
- Use Dockerfile and docker compose to run the app locally in a container. "docker compose up --build" locates the docker-compose file, which will build the image using the Dockerfile, and run it in a local container. While working locally it might be useful to know these commands:
  - Clear Docker cache of local images -> "docker builder prune";
  - List all running Docker container on the machine -> "docker ps";
- At this point the app should run correctly locally, and accessible at a local port like "curl localhost:8080/weather/forecast" in our example;
- To deploy the image to EB, a .NET app needs to be published first:
  - either in Visual Studio Code or with "dotnet publish", publish the app in a folder on your local machine. The output will contain the compiled files and dependencies required to run the project (like the main dll file);
  - at the root of that output folder, you also need to have the Dockerfile / docker compose files. The same docker compose file used for running the app locally can be used. However, the Dockerfile needs to be changed a little bit to remove the "build" phase. When running locally, the docker image was taking the source code, and building + running it. But EB requires that we upload the published file that are already built, so we need to remove that step from the Dockerfile. See the 2 Dockerfiles for reference.
- In EB, create a new application;
- Then create an environment that is using the Docker platform;
- Zip the published output + docker files and upload the bundle to EB;
- The app should be accessible using the environment domain name (ex: http://lb-weather-api-env.eba-uf9ndweg.ca-central-1.elasticbeanstalk.com:8080/weather/forecast).
It is also available using the EC2 public IP if it exists.
Troubleshooting:
- if the deployment was successful but the app remains unreachable, check the security group for the EC2 instance. It needs to have a rule allowing inbound request on the expected port (ex: 8080). 