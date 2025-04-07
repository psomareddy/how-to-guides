# Instrumentation Tutorial with Spring PetClinic Sample Application


## 1. Understanding the Spring Petclinic application with a few diagrams
<a href="https://speakerdeck.com/michaelisvy/spring-petclinic-sample-application">See the presentation here</a>

## 2. Build PetClinic Package
PetClinic is a [Spring Boot](https://spring.io/guides/gs/spring-boot) application built using [Maven](https://spring.io/guides/gs/maven/) or [Gradle](https://spring.io/guides/gs/gradle/). You can build a jar file and run it from the command line (it should work just as well with Java 11 or newer):


Download and Install spring-petclinic application
```cmd
git clone https://github.com/spring-projects/spring-petclinic.git
```

```cmd
cd spring-petclinic
.\mvnw package -Dmaven.test.skip
```

Build artifact are built is target/spring-petclinic-{version}-SNAPSHOT.jar 

## 3. Download Open Telemetry Java Instrumentation 

Execute the following powershell script to download the Otel Java Instrumentation file (splunk-otel-javaagent.jar)
```cmd
Invoke-WebRequest -Uri https://github.com/signalfx/splunk-otel-java/releases/latest/download/splunk-otel-javaagent.jar -OutFile splunk-otel-javaagent.jar
```

## 4. Run PetClinic Application (locally on Windows) with OpenTelemetry APM Instrumentation

Create a startup script

```cmd
notepad start-app.cmd
```

Copy and paste the following contents into the **start-app**.cmd file

```cmd
set OTEL_SERVICE_NAME=petclinic
set OTEL_RESOURCE_ATTRIBUTES=deployment.environment=dev,version=1.1.0
set OTEL_EXPORTER_OTLP_ENDPOINT=http://localhost:4318

set JAVA_OPTIONS="-javaagent:.\splunk-otel-javaagent.jar"

java %JAVA_OPTIONS% -jar .\target\spring-petclinic-3.4.0-SNAPSHOT.jar
```

Execute start-app.cmd

```cmd
start-app.cmd
```

You can then access petclinic here: http://localhost:8080/

## 5. Enable Browser (Real User Monitoring) Instrumentation

The Spring PetClinic application uses a single HTML page as the "layout" page, that is reused across all pages of the
application. This is the perfect location to insert the Splunk RUM Instrumentation Library as it will be loaded in all
pages automatically

Let’s then edit the layout page:

```cmd
notepad src\main\resources\templates\fragments\layout.html
```

Paste the below snippet within the HTML **head** section after all the **meta** tags
```javascript
<script src="https://cdn.signalfx.com/o11y-gdi-rum/latest/splunk-otel-web.js" crossorigin="anonymous"></script>
<script>
    SplunkRum.init({
        realm: "us1",
        rumAccessToken: "REPLACE_WITH_YOUR RUM_TOKEN",
        applicationName: "petclinic",
        deploymentEnvironment: "dev"
    });
</script>
```

Rebuild the deployment jar file and then restart the **start-app.cmd**

```cmd
.\mvnw package -Dmaven.test.skip
```
