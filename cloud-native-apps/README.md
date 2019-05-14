# Demo Cloud Native Apps with Java

## Slides para introdução ao tema...

* [OpenShift - Technical Overview (very low level)](https://docs.google.com/presentation/d/1nwojcNFjOXiRkBRDIiHyWKxB5nyAk0_BWLhHJK2wc0w/edit#slide=id.gb6f3e2d2d_2_213)
* [Inove na velocidade do seu negócio com Openshift e uma estratégia DevOps](https://docs.google.com/presentation/d/1LzDT0TK7PJOsLNPipRNlQcjm0ecikLD6FKXF5lXinEs/edit#slide=id.g17cadb6f71_1_0)
* [Quarkus Supersonic, Subatomic Java](https://docs.google.com/presentation/d/1bHAdgUs2k0gIyt1iySWuliNqbnHXCJOFXkIVOA5-fcs/edit#slide=id.g51e9d14dc4_2_15)
* [Quarkus Supersonic, Subatomic Java 2](https://docs.google.com/presentation/d/1z-of3R2oSBgMWM1teP15RnOd1npNz5NKfPMoMxy2bHY/edit#slide=id.g5298ff92f4_0_0)

### Slides específicos para Docker e Kubernetes

* [Docker for Java Developers](http://redhat.slides.com/rbenevid/docker4devs#/)
* [Tuning JVM for Linux Containers](https://docs.google.com/presentation/d/1SSf5BX22TwAMwCgGtjKdACvvztXlXEW9XG3OttNsfGg/edit#slide=id.g1e56d4b0f2_0_260)
* [Kubernetes for Developers](https://docs.google.com/presentation/d/1A1_3BqWnDu6gFi7JYuCpMeYVzShnPrQZmGiamKcBz84/edit#slide=id.g12c8aac1e6_0_0)

## Pre-req

* CodeReady Containers (former CDK, minishift) ou acesso à um Cluster na nuvem
* OpenJDK 8
* Maven >= 3.6
   > make sure your `~/.m2/settings.xml` have a **active profile** with `https://repo.maven.apache.org/maven2/` repository declared!!!
* VSCode (with Red Hat Java plugin)
* Docker engine ou Podman
* [GraalVM](https://www.graalvm.org/)
  * You need to set `GRAALVM_HOME` before executing this demo
* Packages zlib-devel, gcc and glibc-devel
  * sudo dnf install sudo dnf install gcc glibc-devel zlib-devel
* If running this demo in a local VM, install Nexus (with persistence) to avoid 3G/4G usage during the demo

> Try to run this demo at least once so maven can save all dependencies necessary in its local repository or in Nexus.

## 1) [Quarkus](quarkus.io)

* [Guia oficial](https://quarkus.io/guides/getting-started-guide)

* [Live DevNation demo](https://www.youtube.com/watch?v=7G_r1iyrn2c)

### 1.1) Bootstrap a new project using Quarkus Maven archetype

```bash
# Create dirs
mkdir -p /tmp/dev/demos/quarkus/getting-started && \
cd /tmp/dev/demos/quarkus/getting-started

# Check if you are in the correct path
pwd

# Create app
mvn io.quarkus:quarkus-maven-plugin:0.14.0:create \
    -DprojectGroupId=org.acme \
    -DprojectArtifactId=getting-started \
    -DclassName="org.acme.quickstart.GreetingResource" \
    -Dpath="/hello" \
    -Dextensions="resteasy-jsonb"

# open with VSCode
code .
```

Now show the file structure generate from the maven command.

### 1.2) Start your app in dev mode (live reload!!!)

Open your favorite terminal or use VSCode integrated terminal (``Ctrl + ` `` in VSCode) and execute:

```bash
./mvnw clean compile quarkus:dev
```

> Show the time necessary to start quarkus

Open your browser at [http://localhost:8080/hello](http://localhost:8080/hello)

Change the value returned from hello method to show hot reload:

```java
  @GET
  @Produces(MediaType.TEXT_PLAIN)
  public String hello() {
      return "hello <customer>";
  }
```

> Replace 'hello \<customer\>' using the name of your customer. For example, 'hello nasa'.

Now, go back to your browser and update hello enpoint page to show the hot reload.

Create a new endpoint:

```java
  @GET
  @Produces(MediaType.TEXT_PLAIN)
  @Path("/serverinfo")
  public String serverInfo(@Context HttpServletRequest req) {
      return "hello from quarkus running at: "
          + req.getServerName() + " - " +  req.getLocalAddr();
  }
```

Now, open your browser at [http://localhost:8080/hello/serverinfo](http://localhost:8080/hello/serverinfo)

### 1.3) Injection

Create `GreetingService.java` running the following command:

```java
cat <<EOF > src/main/java/org/acme/quickstart/GreetingService.java
package org.acme.quickstart;

import javax.enterprise.context.ApplicationScoped;

@ApplicationScoped
public class GreetingService {

    public String greeting(String name) {
        return "hello " + name;
    }
}
EOF
```

Update `GreetingResource` endpoint:

```java
...

@Path("/hello")
public class GreetingResource {

    @Inject
    GreetingService greetingService;

    @GET
    @Produces(MediaType.TEXT_PLAIN)
    public String hello() {
        return this.greetingService.greeting("customer");
    }

    @GET
    @Produces(MediaType.TEXT_PLAIN)
    @Path("/serverinfo")
    public String serverInfo(@Context HttpServletRequest req) {
        return "hello from quarkus running at: " + req.getServerName() + " - " + req.getLocalAddr();
    }
}
```

### 1.4) Integration Testing

Update `GreetingResourceTest` so it can pass in the test. To open, in VSCode, press `Crtl + p` and then write `GreetingResourceTest`

First, execute the test to show it won't pass. Click on `Run Test`

> It may be necessary to modified the file and save it before running the test. It seems to be a bug in this release.

Now update it as follow:

```java
    @Test
    public void testHelloEndpoint() {
        given()
          .when().get("/hello")
          .then()
             .statusCode(200)
             .body(is("hello <customer>"));
    }
```

Save the file and execute the test clicking on `Run Test` right above the method signature.

### 1.5) List and install extensions

#### List all extensions available

```bash
mvn quarkus:list-extensions
# or
gradle list-extensions
```

#### Install OpenAPI extension and Health Check

Stop quarkus server before adding these extensions.

> You need to restart quarkus every time a new extensions is added

```bash
# Add extensions
./mvnw quarkus:add-extension -Dextensions="smallrye-openapi"
./mvnw quarkus:add-extension -Dextensions="smallrye-health"

# Run Quarkus
./mvnw clean compile quarkus:dev

# Open openapi
curl http://localhost:8080/openapi
```

Open [http://localhost:8080/swagger-ui](http://localhost:8080/swagger-ui)

Open [http://localhost:8080/health](http://localhost:8080/health)

> [Official extension page](https://quarkus.io/extensions/)

### 1.6) Application Build (Java version)

#### Maven build

```bash
./mvnw package -DskipTests
ls -lah target/
```

#### Local execution

```bash
java -jar target/getting-started-1.0-SNAPSHOT-runner.jar
```

> Notice the startup time

Get java PID:

```bash
JAVA_DEMO_PID=$(ps aux | grep getting-started | awk '{ print $2}' | head -1)
```

Now check how much resource is been used by this java process:  

Linux:

```bash
ps -o pid,rss,cmd -p $JAVA_DEMO_PID | awk 'NR>1 {$2=int($2/1024)"M";}{ print;}'
```

Mac:

```bash
ps -x -o rss,vsz,command | grep $JAVA_DEMO_PID | awk '{$2=int($2/1024)"M";}{ print;}'
```

#### Docker container (OPTIONAL)

Now let's create our container image:

> Before running these commands, make sure quarkus is *not* running

```bash
docker build -f src/main/docker/Dockerfile.jvm -t quarkus/getting-started-jvm .
docker run -i --rm -p 8080:8080 quarkus/getting-started-jvm
docker stats
```

#### Deploying non-native on Openshift/Kubernetes (OPTIONAL)

##### Binary build (OPTIONAL)
  
```bash
oc new-project quarkus-demo

oc new-build --name=quarkus-jvm-demo \
   --image-stream=redhat-openjdk18-openshift:1.4 \
   --env="JAVA_APP_JAR=getting-started-1.0-SNAPSHOT-runner.jar"
   --binary=true -n quarkus-demo

oc start-build quarkus-jvm-demo --from-dir=./target/ocp-build-input/ -n quarkus-demo
oc new-app quarkus-jvm-demo -n quarkus-demo
oc expose svc/quarkus-jvm-demo -n quarkus-demo
```

##### Source to Image (S2I) (OPTIONAL)

Create a new github repo

```bash
git init
git add . --all
git commit -m "first commit"
git remote add origin https://github.com/<user>/quarkus-demo.git
git push -u origin master
```

Java OpenJDK 8 Image Builder
> **NOT WORKING (https://github.com/quarkusio/quarkus-images/issues/13#)!**

* From Catalog -> Java OpenJDK 8 Builder
* Habilitar incremental build

```yaml
strategy:
  sourceStrategy:
    incremental: true
    from:
    ...
```

### 1.7) Native packaging

```bash
# Choose only one option below:

# 1) Build using docker
./mvnw package -Pnative -DskipTests -Dnative-image.container-runtime=docker

# 2) Build using podman
./mvnw package -Pnative -DskipTests -Dnative-image.container-runtime=podman

# 3) Build using local graalvm
./mvnw package -Pnative -DskipTests

ls -lah target/
file target/getting-started-1.0-SNAPSHOT-runner
./target/getting-started-1.0-SNAPSHOT-runner
```

> Notice the startup time!!! **COMPARE WITH JAVA VERSION**

Get native PID:

```bash
JAVA_DEMO_PID=$(ps aux | grep getting-started | awk '{ print $2}' | head -1)
```

Now check how much resource is been used by this java process:

Linux:

```bash
ps -o pid,rss,cmd -p $JAVA_DEMO_PID | awk 'NR>1 {$2=int($2/1024)"M";}{ print;}'
```

Mac:

```bash
ps -x -o rss,vsz,command | grep $JAVA_DEMO_PID | awk '{$2=int($2/1024)"M";}{ print;}'
```

#### Native Docker container (Optional)

Now let's create our container image:

> Before running these commands, make sure quarkus is *not* running

```bash
docker build -f src/main/docker/Dockerfile.native -t quarkus/getting-started .
docker run -i --rm -p 8080:8080 quarkus/getting-started
docker stats
```

#### Native build on Openshift/Kubernetes (Optional)

##### Binary Build

```bash
oc new-project quarkus-demo

oc new-build quay.io/redhat/ubi-quarkus-native-runner \
    --binary \
    --name=quarkus-native-demo \
    -n quarkus-demo

oc start-build quarkus-native-demo \
    --from-file=./target/getting-started-1.0-SNAPSHOT-runner \
    --follow \
    -n quarkus-demo

oc new-app quarkus-native-demo -n quarkus-demo

oc expose svc/quarkus-native-demo -n quarkus-demo
```

##### Source to Image (S2I)

```bash
oc new-project quarkus-demo

# Create BuildConfig
oc new-app quay.io/quarkus/centos-quarkus-native-s2i~{YOUR-QUARKUS-CODE-REPO-URL} \
    --name=quarkus-app \
    -n quarkus-demo

oc expose svc/quarkus-native-demo
```
