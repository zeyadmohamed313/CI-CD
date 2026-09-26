# Build and Test the Application: Practical Guide

This application needs Java 21 or newer. This guide shows exactly how to build, test, run, and package it.

## Start here: run these commands first

Copy this complete block into your terminal before running any Maven command in this project:

```bash
cd "/home/zeyad/work/Study/SideProjects/CI CD"

# Use the installed JDK 25 in this terminal. JDK 25 can build this Java 21 project.
export JAVA_HOME=/usr/lib/jvm/java-1.25.0-openjdk-amd64
export PATH="$JAVA_HOME/bin:$PATH"

# One-time repair for build files created by Docker with another owner.
if [ -d target ]; then sudo chown -R "$USER":"$USER" target; fi

java -version
mvn -version
mvn clean verify
```

The `java -version` and `mvn -version` output must show Java 25 (or any version 21 or newer). The last command must finish with `BUILD SUCCESS`.

## What “build” means

When you build this Spring Boot project, Maven does four useful jobs:

1. Compiles the Java code.
2. Runs the automated tests.
3. Creates one runnable `.jar` file.
4. Places the result in the `target` folder.

The file you will build is:

```text
target/cicd-learning-app-0.0.1-SNAPSHOT.jar
```

## Step 1: Open a terminal in the project folder

```bash
cd "/home/zeyad/work/Study/SideProjects/CI CD"
```

Check that the important files are there:

```bash
ls
```

You should see at least:

```text
pom.xml  src  Dockerfile  README.md
```

## Step 2: Confirm Java and Maven

```bash
java -version
mvn -version
```

Both commands should show Java `21` or a newer version. Maven uses the Java version shown in its output, so this is worth checking before every first build on a new computer. For this Ubuntu machine, use the command block at the top of this guide first. The project still produces Java 21-compatible code.

## Step 3: Make a fresh build

Run this command:

```bash
mvn clean verify
```

What each part means:

- `clean` deletes old files from the `target` folder. This avoids accidentally using an old build.
- `verify` compiles the application, runs tests, creates the runnable JAR, and checks that the build is valid.

At the end, you want to see:

```text
BUILD SUCCESS
```

Check that Maven made the JAR:

```bash
ls -lh target/cicd-learning-app-0.0.1-SNAPSHOT.jar
```

## Step 4: Run the JAR you built

Start the application with the JAR, not Maven:

```bash
java -jar target/cicd-learning-app-0.0.1-SNAPSHOT.jar
```

Keep this terminal open. Spring Boot starts a small web server on port `8080`.

Open a **second** terminal in the same folder and test the greeting API:

```bash
curl 'http://localhost:8080/api/greeting?name=Zeyad'
```

Expected response:

```json
{"message":"Hello, Zeyad!"}
```

Test the health endpoint too:

```bash
curl http://localhost:8080/actuator/health
```

Expected response contains:

```json
{"status":"UP"}
```

When finished, return to the first terminal and press `Ctrl+C` to stop the application.

## Step 5: Understand the automated test

The test lives here:

```text
src/test/java/com/zeyad/cicd/GreetingControllerTest.java
```

It calls the greeting endpoint and checks that `name=Zeyad` returns `Hello, Zeyad!`.

To run only the tests, without first deleting `target`, use:

```bash
mvn test
```

To run only this one test:

```bash
mvn -Dtest=GreetingControllerTest test
```

## Step 6: Make a code change and rebuild

Open this file in your editor:

```text
src/main/java/com/zeyad/cicd/GreetingController.java
```

For a safe experiment, change this text:

```java
"Hello, " + name + "!"
```

to:

```java
"Hi, " + name + "!"
```

Now run:

```bash
mvn clean verify
```

The test should fail, because it still expects `Hello`. This is good: it proves the test is protecting the behaviour.

Update the expected text in `GreetingControllerTest.java` from `Hello, Zeyad!` to `Hi, Zeyad!`, then run again:

```bash
mvn clean verify
```

It should now succeed. You have completed a normal developer cycle: **change → test fails → update test → build succeeds**.

## Step 7: Build a Docker image

Docker packages the application and Java runtime together. That means the app can run on another machine even if Maven is not installed there.

Build the image:

```bash
docker build -t cicd-learning-app:local .
```

The final `.` means “use the current folder as the Docker build context”.

See the image you created:

```bash
docker images cicd-learning-app
```

## Step 8: Run and test the Docker image

Start the container:

```bash
docker run --rm -p 8080:8080 cicd-learning-app:local
```

Explanation:

- `--rm` removes the temporary container when you stop it.
- `-p 8080:8080` connects your computer's port 8080 to the application's port 8080 inside Docker.
- `cicd-learning-app:local` is the image name you made in Step 7.

In a second terminal, call the endpoints again:

```bash
curl 'http://localhost:8080/api/greeting?name=Zeyad'
curl http://localhost:8080/actuator/health
```

Press `Ctrl+C` in the Docker terminal to stop it.

## Step 9: What to run before every Git push

Use this short checklist:

```bash
mvn clean verify
git status
git add .
git commit -m "Describe your change"
git push
```

The first command is the same check that the CI workflow runs on GitHub. Running it locally catches mistakes before you upload your code.

## Common problems

### `mvn` uses the wrong Java version

Run:

```bash
mvn -version
```

If it does not show Java 21 or newer, set your `JAVA_HOME` to a Java 21-or-newer installation and open a new terminal. For the current terminal, the pattern is:

```bash
export JAVA_HOME=/path/to/your/jdk-21-or-newer
export PATH="$JAVA_HOME/bin:$PATH"
java -version
mvn -version
```

For example, if Ubuntu has Java 25 installed at `/usr/lib/jvm/java-1.25.0-openjdk-amd64`, use:

```bash
export JAVA_HOME=/usr/lib/jvm/java-1.25.0-openjdk-amd64
export PATH="$JAVA_HOME/bin:$PATH"
```

Java 25 is newer than Java 21 and works for this project.

### Maven cannot delete a file in `target`

This means old build output was created by a different user (for example, by Docker). Fix the ownership of build output, then build again:

```bash
sudo chown -R "$USER":"$USER" target
mvn clean verify
```

`target` only contains generated build files; it does not contain your source code.

### Port 8080 is already in use

Another application is already running. Stop the earlier Spring Boot process with `Ctrl+C`, or use a different port:

```bash
java -jar target/cicd-learning-app-0.0.1-SNAPSHOT.jar --server.port=8081
```

Then use `http://localhost:8081` in the `curl` commands.

### Maven build fails after your change

Read the first error in the terminal. Usually it tells you the file and line number. You can always return to a clean build with:

```bash
mvn clean verify
```

## How this connects to CI/CD

The same `mvn verify` command is in the CI workflow. GitHub runs it whenever you push to `main` or open a pull request. After you understand the local build, continue to [GITHUB-SETUP-COMMANDS.md](GITHUB-SETUP-COMMANDS.md) to upload the project and practise the GitHub workflows.
