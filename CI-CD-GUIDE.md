# CI/CD Beginner Guide

This project is a small Spring Boot application used to practise CI/CD safely.

First learn how to build and test the application locally in [BUILD-AND-TEST-GUIDE.md](BUILD-AND-TEST-GUIDE.md). For a command-by-command GitHub checklist, see [GITHUB-SETUP-COMMANDS.md](GITHUB-SETUP-COMMANDS.md).

## CI/CD in simple words

**CI (Continuous Integration)** means: whenever you change code, a robot checks that the application still works.

**CD (Continuous Delivery)** means: when you decide a version is ready, a robot creates a release-ready Docker image for you.

In this project, GitHub Actions is the robot.

## Before you start

Install these on your computer:

- Java 21 or newer
- Maven
- Git
- Docker (optional at first, but useful for learning)

Before running Maven on this Ubuntu machine, run the setup commands at the top of [BUILD-AND-TEST-GUIDE.md](BUILD-AND-TEST-GUIDE.md). Then check Java:

```bash
java -version
```

You should see version `21` (or a newer version).

## 1. Run the application

From this folder, run:

```bash
mvn spring-boot:run
```

Leave that terminal open. Open a second terminal and try:

```bash
curl 'http://localhost:8080/api/greeting?name=Zeyad'
```

Expected result:

```json
{"message":"Hello, Zeyad!"}
```

Also try the health check:

```bash
curl http://localhost:8080/actuator/health
```

Stop the application with `Ctrl+C`.

## 2. Run the check locally

This is the important command:

```bash
mvn verify
```

It compiles the application and runs its tests. If it finishes with `BUILD SUCCESS`, your code is ready for CI.

## 3. Put the project on GitHub

Create a new empty repository on GitHub. Then run these commands in this folder, replacing the URL with your repository URL:

```bash
git init -b main
git add .
git commit -m "Initial CI/CD learning app"
git remote add origin https://github.com/YOUR-USERNAME/YOUR-REPOSITORY.git
git push -u origin main
```

Go to the **Actions** tab in GitHub. You should see a workflow called **Continuous Integration** running.

## What CI does

The file `.github/workflows/ci.yml` tells GitHub what to do whenever you:

- push to `main`, or
- open or update a pull request.

It does three checks:

1. Sets up Java 21.
2. Runs `mvn verify`.
3. Builds a Docker image, but does not publish it.

If a check fails, GitHub marks the workflow red. Fix the code, push again, and GitHub tries again.

## 4. Practise CI with a small change

Create a branch:

```bash
git checkout -b change-greeting
```

Change `Hello` to `Hi` in `GreetingController.java`, then run:

```bash
mvn verify
git add .
git commit -m "Change greeting"
git push -u origin change-greeting
```

On GitHub, open a pull request. Watch CI run in the Actions tab. When it is green, merge the pull request into `main`.

## 5. Create your first release

After the CI workflow is green on `main`, make a release tag:

```bash
git checkout main
git pull
git tag v0.1.0
git push origin v0.1.0
```

This starts the **Continuous Delivery** workflow in `.github/workflows/cd.yml`.

It runs the tests again and publishes a Docker image to GitHub Container Registry (GHCR). Find it on your repository page under **Packages**.

The image name will be:

```text
ghcr.io/YOUR-USERNAME/YOUR-REPOSITORY:v0.1.0
```

## 6. Run the released image

After the release workflow succeeds:

```bash
docker pull ghcr.io/YOUR-USERNAME/YOUR-REPOSITORY:v0.1.0
docker run --rm -p 8080:8080 ghcr.io/YOUR-USERNAME/YOUR-REPOSITORY:v0.1.0
```

Then open a second terminal and call the greeting endpoint again.

## If something goes wrong

| Problem | First thing to check |
| --- | --- |
| `mvn` says Java is too old | Run the setup block in [BUILD-AND-TEST-GUIDE.md](BUILD-AND-TEST-GUIDE.md) to select Java 25 for this terminal. |
| CI is red | Open the failed GitHub Actions run and read the first error message. |
| Image does not publish | Check the Continuous Delivery run and make sure you pushed a tag beginning with `v`, such as `v0.1.0`. |
| `docker pull` is denied | The package may still be private. Make it public in the package settings, or sign in with `docker login ghcr.io`. |

## What comes next?

Right now, CD creates a tested Docker image. It does **not** deploy the image to a website or cloud server yet. Once this feels comfortable, choose a deployment target (for example Render, Railway, Fly.io, AWS, or your own VPS) and add one deployment step.
