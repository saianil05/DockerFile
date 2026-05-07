# DockerFile
Production Level DockerFile

# Production Dockerfile - Complete Word by Word Explanation

---

# Full Production Dockerfile

```dockerfile
# =========================================================
# Production Grade Dockerfile for Java Spring Boot App
# =========================================================

FROM maven:3.9.6-eclipse-temurin-21 AS builder

LABEL maintainer="devops-team@company.com"
LABEL application="customer-service"

WORKDIR /build

COPY pom.xml .

RUN mvn dependency:go-offline

COPY src ./src

RUN mvn clean package -DskipTests

FROM eclipse-temurin:21-jre

RUN groupadd -g 1001 appgroup && \
    useradd -u 1001 -g appgroup -m -s /bin/bash appuser

ENV APP_HOME=/opt/app
ENV APP_NAME=customer-service
ENV JAVA_OPTS="-Xms512m -Xmx1024m"
ENV TZ=UTC

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    curl \
    dumb-init && \
    rm -rf /var/lib/apt/lists/*

RUN mkdir -p ${APP_HOME}

WORKDIR ${APP_HOME}

COPY --from=builder /build/target/*.jar app.jar

RUN chown -R appuser:appgroup ${APP_HOME}

USER appuser

HEALTHCHECK --interval=30s \
             --timeout=5s \
             --start-period=40s \
             --retries=3 \
CMD curl -f http://localhost:8080/actuator/health || exit 1

EXPOSE 8080

ENTRYPOINT ["dumb-init", "--"]

CMD ["sh", "-c", "java $JAVA_OPTS -jar app.jar"]
```

---

# COMPLETE EXPLANATION

---

# LINE 1

```dockerfile
FROM maven:3.9.6-eclipse-temurin-21 AS builder
```

## What is FROM?

`FROM` means:

```text
Start building this Docker image using another already existing image.
```

Docker images are layered.

This line tells Docker:

```text
Use Maven + Java image as the base image.
```

---

## What is maven?

`maven` is the image name.

This image already contains:

* Linux OS
* Maven installed
* Java installed

---

## What is : ?

```dockerfile
:
```

Colon separates:

```text
image-name : image-version
```

---

## What is 3.9.6-eclipse-temurin-21?

This is the image tag/version.

Breakdown:

| Part            | Meaning           |
| --------------- | ----------------- |
| 3.9.6           | Maven version     |
| eclipse-temurin | Java distribution |
| 21              | Java version      |

---

## What is AS?

`AS` gives a name to this build stage.

---

## What is builder?

`builder` is the stage name.

Later we use:

```dockerfile
COPY --from=builder
```

Meaning:

```text
Copy files from the builder stage.
```

---

## Why Multi-stage Build?

Without multi-stage build:

Final image contains:

* Maven
* Source code
* Cache
* Build tools

Problems:

* Large image
* Slow deployment
* Security risk

With multi-stage build:

Final image contains only:

* Java runtime
* Application JAR

Benefits:

* Small image
* Faster deployment
* Better security

---

# LINE 2

```dockerfile
LABEL maintainer="devops-team@company.com"
```

## What is LABEL?

`LABEL` adds metadata to Docker image.

Metadata means:

```text
Information about the image.
```

---

## What is maintainer?

Custom key name.

Stores owner/team information.

---

## What is = ?

Assignment operator.

---

## What is "[devops-team@company.com](mailto:devops-team@company.com)"?

Metadata value.

---

## Why LABEL is useful?

Useful for:

* Automation
* Ownership
* Monitoring
* Auditing

---

# LINE 3

```dockerfile
LABEL application="customer-service"
```

Stores application name.

Useful for:

* Monitoring tools
* Logging
* CI/CD
* Automation

---

# LINE 4

```dockerfile
WORKDIR /build
```

## What is WORKDIR?

Sets working directory inside container.

Equivalent Linux command:

```bash
cd /build
```

---

## What is /build?

Directory path.

If folder doesn't exist:
Docker automatically creates it.

---

# LINE 5

```dockerfile
COPY pom.xml .
```

## What is COPY?

Copies files from:

* local machine

To:

* Docker image

---

## What is pom.xml?

Maven project file.

Contains:

* Dependencies
* Plugins
* Build settings
* Project details

---

## What is . ?

`.` means:

```text
Current directory
```

Current directory is:

```text
/build
```

because of WORKDIR.

---

# LINE 6

```dockerfile
RUN mvn dependency:go-offline
```

## What is RUN?

Executes command while building image.

---

## What is mvn?

Maven command.

---

## What is dependency:go-offline?

Downloads all Maven dependencies.

Examples:

* Spring Boot
* Hibernate
* Jackson
* Logging libraries

---

## Why done separately?

Docker layer caching optimization.

If source code changes:

Dependencies need not download again.

Benefits:

* Faster builds
* Faster CI/CD

---

# LINE 7

```dockerfile
COPY src ./src
```

## What is src?

Application source folder.

Contains:

* Java classes
* Resources
* Configuration files

---

## What is ./src?

Destination folder inside container.

---

# LINE 8

```dockerfile
RUN mvn clean package -DskipTests
```

## What is clean?

Deletes old build files.

Equivalent:

```text
Remove old target folder
```

---

## What is package?

Builds application.

Creates:

```text
target/app.jar
```

---

## What is -DskipTests?

Skips test execution.

---

## Why skip tests?

Production CI/CD pipelines already run tests.

Skipping tests:

* Faster Docker build

---

# LINE 9

```dockerfile
FROM eclipse-temurin:21-jre
```

Starts second stage.

This becomes final runtime image.

---

## What is eclipse-temurin?

Java distribution.

---

## What is 21?

Java version.

---

## What is jre?

Java Runtime Environment.

Contains only:

* Java runtime

Does NOT contain:

* Compiler
* Maven
* Build tools

---

## Why use JRE?

Benefits:

* Smaller image
* Better security
* Faster startup

---

# LINE 10

```dockerfile
RUN groupadd -g 1001 appgroup
```

## What is groupadd?

Linux command to create group.

---

## What is -g?

Specifies Group ID.

---

## What is 1001?

Group ID.

Linux internally identifies groups using numbers.

---

## Why 1001?

Linux ranges:

| Range | Purpose      |
| ----- | ------------ |
| 0     | root         |
| 1-999 | system users |
| 1000+ | normal users |

1001 is safe production UID/GID.

---

## What is appgroup?

Group name.

---

# LINE 11

```dockerfile
useradd -u 1001 -g appgroup -m -s /bin/bash appuser
```

## What is useradd?

Creates Linux user.

---

## What is -u?

Specifies User ID.

---

## What is 1001?

User ID.

---

## Why fixed UID important?

Helps avoid:

* Permission issues
* Kubernetes volume problems
* Shared storage issues

---

## What is -g?

Assigns primary group.

---

## What is appgroup?

Primary group name.

---

## What is -m?

Creates home directory.

Example:

```text
/home/appuser
```

---

## What is -s?

Specifies shell.

---

## What is /bin/bash?

Default shell.

---

## What is appuser?

Username.

---

# LINE 12

```dockerfile
ENV APP_HOME=/opt/app
```

## What is ENV?

Creates environment variable.

Equivalent Linux:

```bash
export APP_HOME=/opt/app
```

---

## What is APP_HOME?

Variable name.

---

## What is /opt/app?

Variable value.

Application installation directory.

---

# LINE 13

```dockerfile
ENV APP_NAME=customer-service
```

Stores application name.

Useful for:

* Logging
* Monitoring
* Automation

---

# LINE 14

```dockerfile
ENV JAVA_OPTS="-Xms512m -Xmx1024m"
```

## What is JAVA_OPTS?

Java runtime options.

---

## What is -Xms512m?

Initial Java heap memory.

512 MB.

---

## What is -Xmx1024m?

Maximum Java heap memory.

1024 MB = 1 GB.

---

# LINE 15

```dockerfile
ENV TZ=UTC
```

## What is TZ?

Timezone variable.

---

## Why UTC?

Production best practice.

Avoids timezone issues.

---

# LINE 16

```dockerfile
RUN apt-get update
```

## What is apt-get?

Ubuntu/Debian package manager.

---

## What is update?

Downloads latest package index.

Equivalent:

```bash
sudo apt update
```

---

# LINE 17

```dockerfile
apt-get install -y --no-install-recommends
```

## What is install?

Installs packages.

---

## What is -y?

Automatically answer YES.

---

## What is --no-install-recommends?

Installs only required packages.

Avoids unnecessary packages.

Benefits:

* Smaller image
* Better security

---

# LINE 18

```dockerfile
curl
```

Linux utility.

Used for:

* API calls
* Health checks
* Debugging

---

# LINE 19

```dockerfile
dumb-init
```

Tiny init system.

Fixes:

* Zombie processes
* Signal handling
* Graceful shutdown

Very important in Kubernetes.

---

# LINE 20

```dockerfile
rm -rf /var/lib/apt/lists/*
```

## What is rm?

Delete/remove files.

---

## What is -r?

Recursive delete.

---

## What is -f?

Force delete.

---

## What is /var/lib/apt/lists/*?

APT package cache.

Deleting it reduces image size.

---

# LINE 21

```dockerfile
RUN mkdir -p ${APP_HOME}
```

## What is mkdir?

Creates directory.

---

## What is -p?

Creates parent folders automatically.

---

## What is ${APP_HOME}?

Uses environment variable value.

Expands to:

```text
/opt/app
```

---

# LINE 22

```dockerfile
WORKDIR ${APP_HOME}
```

Equivalent:

```bash
cd /opt/app
```

---

# LINE 23

```dockerfile
COPY --from=builder /build/target/*.jar app.jar
```

## What is --from=builder?

Copy from previous build stage.

---

## What is /build/target/*.jar?

Source path.

Built JAR location.

---

## What is *.jar?

Wildcard.

Matches any JAR file.

---

## What is app.jar?

Destination filename.

---

## Why important?

Final image contains ONLY:

* app.jar

NOT:

* Maven
* Source code
* Cache

Huge optimization.

---

# LINE 24

```dockerfile
RUN chown -R appuser:appgroup ${APP_HOME}
```

## What is chown?

Changes ownership.

---

## What is -R?

Recursive.

Applies to all files/subfolders.

---

## What is appuser:appgroup?

New owner and group.

---

# LINE 25

```dockerfile
USER appuser
```

Runs container as:

* non-root user

Security best practice.

---

# LINE 26

```dockerfile
HEALTHCHECK --interval=30s
```

Docker health monitoring.

---

## What is --interval?

How often to run health check.

30 seconds.

---

## What is --timeout?

Maximum wait time.

5 seconds.

---

## What is --start-period?

Initial delay before health checks.

40 seconds.

---

## What is --retries?

Retry attempts before marking unhealthy.

---

# LINE 27

```dockerfile
CMD curl -f http://localhost:8080/actuator/health || exit 1
```

## What is CMD?

Command to execute.

---

## What is curl?

Makes HTTP request.

---

## What is -f?

Fail if HTTP response is:

* 400
* 500

---

## What is localhost?

Container itself.

---

## What is 8080?

Application port.

---

## What is actuator/health?

Spring Boot health endpoint.

---

## What is || exit 1?

If command fails:

Return error code 1.

Docker marks container unhealthy.

---

# LINE 28

```dockerfile
EXPOSE 8080
```

Documents container port.

Application listens on:

* 8080

---

# LINE 29

```dockerfile
ENTRYPOINT ["dumb-init", "--"]
```

## What is ENTRYPOINT?

Main executable.

---

## Why dumb-init?

Handles:

* Shutdown signals
* Child process cleanup
* Graceful shutdown

---

# LINE 30

```dockerfile
CMD ["sh", "-c", "java $JAVA_OPTS -jar app.jar"]
```

## What is sh?

Linux shell.

---

## What is -c?

Execute command string.

---

## What is java?

Starts Java runtime.

---

## What is $JAVA_OPTS?

Expands environment variable.

Example:

```text
-Xms512m -Xmx1024m
```

---

## What is -jar?

Run JAR file.

---

## What is app.jar?

Spring Boot application.

---

# FINAL FLOW

```text
Developer Writes Code
        ↓
Docker Build Starts
        ↓
Maven Builds Application
        ↓
JAR Created
        ↓
Runtime Image Created
        ↓
Container Starts
        ↓
dumb-init starts
        ↓
Java App Starts
        ↓
Healthcheck Monitors App
        ↓
Production Traffic Comes
```

---

# Production Best Practices Used

| Feature           | Benefit            |
| ----------------- | ------------------ |
| Multi-stage build | Small image        |
| Non-root user     | Better security    |
| Healthcheck       | Failure detection  |
| JRE runtime       | Smaller image      |
| dumb-init         | Graceful shutdown  |
| Layer caching     | Faster builds      |
| Cleanup           | Reduced image size |

---

# Docker Build Command

```bash
docker build -t customer-service:v1 .
```

---

# Docker Run Command

```bash
docker run -d -p 8080:8080 customer-service:v1
```
