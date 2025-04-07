## Create Docker Image for PetClinic App

Create the Dockerfile from the snippet below

```properties
FROM openjdk:17

EXPOSE 8080

# Adds the latest version of the Splunk Java agent
ADD --chown=javauser:javauser https://github.com/signalfx/splunk-otel-java/releases/latest/download/splunk-otel-javaagent.jar /opt/splunk-otel-javaagent.jar

WORKDIR /app

COPY ./spring-petclinic/target/spring-petclinic-3.4.0-SNAPSHOT.jar /app/petclinic.jar

ENTRYPOINT [ "java", "-javaagent:/opt/splunk-otel-javaagent.jar",  "-jar","/app/petclinic.jar"]
```

Always re-build the PetClinic package to ensure you have the latest build with all the source code changes built in

```cmd
.\mvnw package -Dmaven.test.skip
```

Execute the following command to build the image
```cmd
docker build -t petclinic-app . -f Dockerfile
```

Create a container from the above image and run it (including required Open Telemetry env variables)

```cmd
docker run --name petclinic -d -p 8080:8080 petclinic-app:latest -e OTEL_SERVICE_NAME="petclinic" -e OTEL_RESOURCE_ATTRIBUTES"deployment.environment=dev" -e OTEL_EXPORTER_OTLP_ENDPOINT="http://host.docker.internal:4318" 
```
