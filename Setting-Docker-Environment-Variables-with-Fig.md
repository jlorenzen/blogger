For the past few months I've been playing around with [Docker](https://www.docker.com/), and so far I've had a ton of fun. The [documentation](https://docs.docker.com/) is excellent, and in one simple [command](https://docs.docker.com/userguide/dockerizing/#hello-world) you can start experimenting. After going through the tutorial, one of my first goals was to figure out the best way to create a Docker image for a [spring-boot](http://projects.spring.io/spring-boot/) service. My initial goals were to make it easy to set environment variables, since our projects follow the [twelve-factor](http://12factor.net/) methodology. One of the several factors we follow is the third factor (III. Config), which recommends storing the config in the environment. While this has many benefits, one of the downsides is the tendency to create a lot of environment variables because it's quick, easy, and well defined. This makes it difficult to configure, test, and run the service. But as we will see, [Fig](http://www.fig.sh/) will not only make it easy to set environment variables, it will also provide many other benefits.

# Docker Image
Let's first start with a pretend service called the logging service. It's a Java service created with spring-boot. Here is a basic Dockerfile:
```
FROM dockerfile/java:oracle-java8
COPY logging-service-0.1.0.jar /data/
EXPOSE 8080
CMD ["java", "-jar", "logging-service-0.1.0.jar"]
```
* [FROM](https://docs.docker.com/reference/builder/#from) - the base image I start with. In this case it's the [dockerfile/java](https://registry.hub.docker.com/u/dockerfile/java) base image with the Oracle JDK 8 tag.
* [COPY](https://docs.docker.com/reference/builder/#copy) - here I copy over the jar so it's present in the image
* [EXPOSE](https://docs.docker.com/reference/builder/#expose) - this tells Docker the container will be listening on port 8080 at runtime
* [CMD](https://docs.docker.com/reference/builder/#cmd) - here I've defined a default command to run which will start the service

Next we need to build the image:
```
docker build --tag="jlorenzen/logging-service:v1" .
```

# Docker Run
Now we could run our new service by executing this command:
```
docker run -dP jlorenzen/logging-service
```
That's great but let's imagine the logging-service requires the following environment variables: `ENV_1` and `ENV_2`. Here is how you would run the service while also setting the environment variables:
```
docker run -dP -e ENV_1=value1 -e ENV_2=value2 jlorenzen/logging-service
```

That's a basic example, but you can image how nasty it could get if your service required a dozen or more environment variables. The `docker run` command also has some other nice options for setting environment variables. For example, when using the `-e` option, if you provide just the name like `-e ENV_1` without a value, than that variables current value will be used. Or you can use the `--env-file` option to specify a file that contains a list of environment variables. While this all works, it's really not enjoyable having to remember all those options and commands. That is where Fig can help. And it not only helps us easily set environment variables, but it also makes creating containers simpler and reproducable by anyone anywhere.

# Fig
Fig is basically a simple utility that wraps Docker making it easier to create and manage Docker containers. In our case we will use it to run our logging-service image and set the environment variables. Here is a simple `fig.yml` file:
```
logging-service:
  image: jlorenzen/logging-service
  ports: 
   - "8080"
  environment:
   - ENV_1
   - ENV_2
```
As you can see I didn't specify any values for the environment variables. That's because I already have them defined in my host using [direnv](http://direnv.net/) and Fig will just automatically use them. So in my case I have a local `.envrc` file that contains the following:
```
export ENV_1=value1
export ENV_2=value2
```
This allows me to set all my environment variables in one place. Here is the command I can use to start the container: `fig up`. That's it! Much simpler than the corresponding docker run command.

# Ideal World
What would be the best of both worlds is if Fig supported the `docker run --env-file` option and that it could read in a file containing `export` commands which is required by direnv. It seems support for the `--env-file` option in Fig is [coming soon](https://twitter.com/jlorenzen/status/553195845135650816), so we are halfway there.
