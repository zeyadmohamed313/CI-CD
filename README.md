# Spring Boot CI/CD Learning App

A tiny Spring Boot API designed to let you practise a complete CI/CD loop without first having to understand a large application.

**New to CI/CD?** First follow the practical [Build and Test guide](BUILD-AND-TEST-GUIDE.md), then read [CI-CD-GUIDE.md](CI-CD-GUIDE.md) and use the copy-and-paste [GitHub setup commands](GITHUB-SETUP-COMMANDS.md).

## What it does

- `GET /api/greeting?name=Zeyad` returns `{"message":"Hello, Zeyad!"}`.
- `GET /actuator/health` reports whether the application is healthy.
- One focused unit test protects the greeting behaviour.

## Run it locally

This project needs Java 21 or newer and Maven 3.6.3 or newer.

On this Ubuntu machine, first run the setup block at the top of [BUILD-AND-TEST-GUIDE.md](BUILD-AND-TEST-GUIDE.md). It selects Java 25 for your terminal and repairs the existing `target` folder permissions.

```bash
mvn spring-boot:run
curl 'http://localhost:8080/api/greeting?name=Zeyad'
curl http://localhost:8080/actuator/health
```

Run the same checks that CI runs:

```bash
mvn verify
```

You can also package and run it as a container:

```bash
docker build -t cicd-learning-app:local .
docker run --rm -p 8080:8080 cicd-learning-app:local
```

## The pipeline

```
Pull request or push to main
            |
            v
    CI: Maven tests + JAR
            |
            v
     CI: Docker image build

Push a version tag, for example v0.1.0
            |
            v
CD: publish image to ghcr.io/<owner>/<repository>
```

`ci.yml` proves that every change compiles, passes tests, and can become a container image. `cd.yml` only runs for version tags, so an image is not released until you deliberately create a version.

This is **continuous delivery**: the pipeline publishes a release-ready image. Deploying that image to a particular cloud provider is intentionally the next lesson, because it needs a target platform and credentials.

## Put the project on GitHub

After creating an empty GitHub repository, run these commands from this directory:

```bash
git init -b main
git add .
git commit -m "Initial Spring Boot CI/CD learning app"
git remote add origin https://github.com/YOUR-USERNAME/YOUR-REPOSITORY.git
git push -u origin main
```

The CI workflow starts on pushes to `main` and on pull requests. GitHub-hosted runners provide Java 21, so your local Java version does not affect the workflow.

## Release your first image

Once CI is green on GitHub, create and push a semantic version tag:

```bash
git tag v0.1.0
git push origin v0.1.0
```

The delivery workflow uses GitHub's built-in `GITHUB_TOKEN`; no registry password is needed. Its `packages: write` permission lets it publish the image to GitHub Container Registry. After the first publish, you can change the package visibility in GitHub if you want to pull it without authentication.

## Suggested exercises

1. Change the greeting text without updating the test, push a branch, and watch CI fail.
2. Fix the test, open a pull request, and merge after CI passes.
3. Tag `v0.1.0` and inspect the image tags in the repository's Packages section.
4. Pull the published image and run it locally:

   ```bash
   docker pull ghcr.io/YOUR-USERNAME/YOUR-REPOSITORY:v0.1.0
   docker run --rm -p 8080:8080 ghcr.io/YOUR-USERNAME/YOUR-REPOSITORY:v0.1.0
   ```

The next natural step is an actual deployment workflow to one chosen target, such as Render, Railway, Fly.io, AWS, or a VPS.
